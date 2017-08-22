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

上面的DAO实现方式的好处在于只依赖于Hibernate API，而无需引入Spring的class。这从非侵入性的角度来看当然是有吸引力的，毫无疑问，这种开发方式会令Hibernate开发人员将会更加自然。

然而，DAO层会抛出Hibernate自有异常`HibernateException`（属于非检查异常，无需显式声明和使用try-catch），但是也意味着调用方会将异常看做致命异常——除非调用方将Hibernate异常体系作为应用的异常体系来处理。而在这种情况下，除非调用方自己来实现一定的策略，否则捕获一些诸如乐观锁失败之类的特定错误是不可能的。对于强烈基于Hibernate的应用程序或不需要对特殊异常处理的应用程序，这种代价可能是可以接受的。

幸运的是，Spring的`LocalSessionFactoryBean`可以通过Hibernate的`SessionFactory.getCurrentSession()`方法为所有的Spring事务策略提供支持，使用`HibernateTransactionManager`返回当前的Spring管理的事务的`Session`。当然，该方法的标准行为仍然是返回与正在进行的JTA事务相关联的当前`Session`（如果有的话）。无论开发者是使用Spring的`JtaTransactionManager`，EJB容器管理事务（CMT）还是JTA，都会适用此行为。

总而言之：开发者可以基于纯Hibernate API来实现DAO，同时也可以集成Spring来管理事务。

