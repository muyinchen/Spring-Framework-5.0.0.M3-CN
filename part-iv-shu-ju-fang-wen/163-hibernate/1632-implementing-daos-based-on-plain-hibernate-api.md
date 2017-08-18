### 16.3.2基于Hibernate API来实现DAO

Hibernate有一个特性称之为上下文会话，在每个Hibernate本身每个事务都管理一个当前的`Session`。这大致相当于Spring每个事务的一个Hibernate`Session`的同步。如下的DAO的实现类就是基于简单的Hibernate API实现的：

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

除了需要在实例中持有`SessionFactory`引用以外，上面的代码风格跟Hibernate文档中的例子十分相近。Spring团队强烈建议使用这种基于实例变量的实现风格，而非守旧的`static HibernateUtil`风格\(总的来说，除非_绝对_必要，否则尽量不要使用`static`变量来持有资源\)。

上面DAO的实现完全符合Spring依赖注入的样式：这种方式可以很好的集成Spring IoC容器，就好像Spring的`HibernateTemplate`代码一样。当然，DAO层的实现也可以通过纯Java的方式来配置（比如在UT中）。简单实例化`ProductDaoImpl`并且调用`setSessionFactory(...)`即可。当然，也可以使用Spring bean来进行注入，参考如下XML配置：

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

