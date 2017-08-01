### 16.2.2Exception translation

When you use Hibernate or JPA in a DAO, you must decide how to handle the persistence technology’s native exception classes. The DAO throws a subclass of a`HibernateException`or`PersistenceException`depending on the technology. These exceptions are all runtime exceptions and do not have to be declared or caught. You may also have to deal with`IllegalArgumentException`and`IllegalStateException`. This means that callers can only treat exceptions as generally fatal, unless they want to depend on the persistence technology’s own exception structure. Catching specific causes such as an optimistic locking failure is not possible without tying the caller to the implementation strategy. This trade-off might be acceptable to applications that are strongly ORM-based and/or do not need any special exception treatment. However, Spring enables exception translation to be applied transparently through the`@Repository`annotation:

```java
@Repository
public class ProductDaoImpl implements ProductDao {

	// class body here...

}
```

```java
<beans>

	<!-- Exception translation bean post processor -->
	<bean class="org.springframework.dao.annotation.PersistenceExceptionTranslationPostProcessor"/>

	<bean id="myProductDao" class="product.ProductDaoImpl"/>

</beans>
```

The postprocessor automatically looks for all exception translators \(implementations of the`PersistenceExceptionTranslator`interface\) and advises all beans marked with the`@Repository`annotation so that the discovered translators can intercept and apply the appropriate translation on the thrown exceptions.

In summary: you can implement DAOs based on the plain persistence technology’s API and annotations, while still benefiting from Spring-managed transactions, dependency injection, and transparent exception conversion \(if desired\) to Spring’s custom exception hierarchies.

