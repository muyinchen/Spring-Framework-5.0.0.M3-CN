### 16.3.2Implementing DAOs based on plain Hibernate API

Hibernate has a feature called contextual sessions, wherein Hibernate itself manages one current`Session`per transaction. This is roughly equivalent to Spring’s synchronization of one Hibernate`Session`per transaction. A corresponding DAO implementation resembles the following example, based on the plain Hibernate API:

```java
public class ProductDaoImpl implements ProductDao {

	private SessionFactory sessionFactory;

	public void setSessionFactory(SessionFactory sessionFactory) {
		this.sessionFactory = sessionFactory;
	}

	public Collection loadProductsByCategory(String category) {
		return this.sessionFactory.getCurrentSession()
				.createQuery("from test.Product product where product.category=?")
				.setParameter(0, category)
				.list();
	}
}
```

This style is similar to that of the Hibernate reference documentation and examples, except for holding the`SessionFactory`in an instance variable. We strongly recommend such an instance-based setup over the old-school`staticHibernateUtil`class from Hibernate’s CaveatEmptor sample application. \(In general, do not keep any resources in`static`variables unless_absolutely_necessary.\)

The above DAO follows the dependency injection pattern: it fits nicely into a Spring IoC container, just as it would if coded against Spring’s`HibernateTemplate`. Of course, such a DAO can also be set up in plain Java \(for example, in unit tests\). Simply instantiate it and call`setSessionFactory(..)`with the desired factory reference. As a Spring bean definition, the DAO would resemble the following:

```java
<beans>

	<bean id="myProductDao" class="product.ProductDaoImpl">
		<property name="sessionFactory" ref="mySessionFactory"/>
	</bean>

</beans>
```

The main advantage of this DAO style is that it depends on Hibernate API only; no import of any Spring class is required. This is of course appealing from a non-invasiveness perspective, and will no doubt feel more natural to Hibernate developers.

However, the DAO throws plain`HibernateException`\(which is unchecked, so does not have to be declared or caught\), which means that callers can only treat exceptions as generally fatal - unless they want to depend on Hibernate’s own exception hierarchy. Catching specific causes such as an optimistic locking failure is not possible without tying the caller to the implementation strategy. This trade off might be acceptable to applications that are strongly Hibernate-based and/or do not need any special exception treatment.

Fortunately, Spring’s`LocalSessionFactoryBean`supports Hibernate’s`SessionFactory.getCurrentSession()`method for any Spring transaction strategy, returning the current Spring-managed transactional`Session`even with`HibernateTransactionManager`. Of course, the standard behavior of that method remains the return of the current`Session`associated with the ongoing JTA transaction, if any. This behavior applies regardless of whether you are using Spring’s`JtaTransactionManager`, EJB container managed transactions \(CMTs\), or JTA.

In summary: you can implement DAOs based on the plain Hibernate API, while still being able to participate in Spring-managed transactions.

  


