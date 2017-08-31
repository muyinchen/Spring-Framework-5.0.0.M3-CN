### 16.4.2基于JPA的EntityManagerFactory和EntityManager来实现DAO

| ![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| :--- |
| 虽然`EntityManagerFactory`实例是线程安全的，但`EntityManager`实例不是。注入的JPA `EntityManager`的行为类似于从JPA Spec中定义的应用程序服务器的JNDI环境中提取的`EntityManager`。它将所有调用委托给当前事务的`EntityManager`\(如果有\);否则，它每个操作返回的都是新创建的`EntityManager`，通过使用不同的`EntityManager`来保证使用时的线程安全。 |

通过注入的方式使用`EntityManagerFactory`或`EntityManager`来编写JPA代码，是不需要依赖任何Spring定义的类的。如果启用了`PersistenceAnnotationBeanPostProcessor`，Spring可以在实例级别和方法级别识别`@PersistenceUnit`和`@PersistenceContext`注解。使用`@PersistenceUnit`注解的纯JPA DAO实现可能如下所示：

```java
public class ProductDaoImpl implements ProductDao {

    private EntityManagerFactory emf;

    @PersistenceUnit
    public void setEntityManagerFactory(EntityManagerFactory emf) {
        this.emf = emf;
    }

    public Collection loadProductsByCategory(String category) {
        EntityManager em = this.emf.createEntityManager();
        try {
            Query query = em.createQuery("from Product as p where p.category = ?1");
            query.setParameter(1, category);
            return query.getResultList();
        }
        finally {
            if (em != null) {
                em.close();
            }
        }
    }
}
```

上面的DAO对Spring的实现是没有任何依赖的，而且很适合与Spring的应用程序上下文进行集成。而且，DAO还可以通过注解来注入默认的`EntityManagerFactory`：

```java
<beans>

    <!-- bean post-processor for JPA annotations -->
    <bean class="org.springframework.orm.jpa.support.PersistenceAnnotationBeanPostProcessor"/>

    <bean id="myProductDao" class="product.ProductDaoImpl"/>

</beans>
```

如果不想明确定义`PersistenceAnnotationBeanPostProcessor`，可以考虑在应用程序上下文配置中使用Spring上下文`annotation-config`XML元素。这样做会自动注册所有Spring标准后置处理器，用于初始化基于注解的配置，包括`CommonAnnotationBeanPostProcessor`等。

```java
<beans>

    <!-- post-processors for all standard config annotations -->
    <context:annotation-config/>

    <bean id="myProductDao" class="product.ProductDaoImpl"/>

</beans>
```

这样的DAO的主要问题是它总是通过工厂创建一个新的`EntityManager`。开发者可以通过请求事务性`EntityManager`（也称为_共享`EntityManager`_，因为它是实际的事务性EntityManager的一个共享的，线程安全的代理）来避免这种情况。

```java
public class ProductDaoImpl implements ProductDao {

    @PersistenceContext
    private EntityManager em;

    public Collection loadProductsByCategory(String category) {
        Query query = em.createQuery("from Product as p where p.category = :category");
        query.setParameter("category", category);
        return query.getResultList();
    }
}
```

`@PersistenceContext`注解具有可选的属性类型，默认值为`PersistenceContextType.TRANSACTION`。此默认值是开发者所需要接收共享的`EntityManager`代理。替代方案`PersistenceContextType.EXTENDED`则完全不同：该方案会返回一个所谓扩展的`EntityManager`，该`EntityManager`不是_线程安全_的，因此不能在并发访问的组件（如Spring管理的单例Bean）中使用。扩展实体管理器仅应用于状态组件中，比如持有会话的组件，其中`EntityManager`的生命周期与当前事务无关，而是完全取决于应用程序。

**方法和实例变量级别注入**

指示依赖注入（例如`@PersistenceUnit`和`@PersistenceContext`）的注解可以应用于类中的实例变量或方法，也就是表达式方法级注入和实例变量级注入。实例变量级注释简洁易用，而方法级别允许进一步处理注入的依赖关系。在这两种情况下，成员的可见性（`public`，`protected`，`private`）并不重要。

类级注解怎么办？

在J2EE平台上，它们用于依赖关系声明，而不是资源注入。

注入的`EntityManager`是由Spring管理的（Spring可以意识到正在进行的事务）。重要的是要注意，因为通过注解进行注入，即使新的DAO实现使用通过方法注入的`EntityManager`而不是`EntityManagerFactory`的注入的，在应用程序上下文XML中不需要进行任何修改。

这种DAO风格的主要优点是它只依赖于Java Persistence API;不需要导入任何Spring的实现类。而且，Spring容器可以识别JPA注解来实现自动的注入和管理。从非侵入的角度来看，这种风格对JPA开发者来说可能更为自然。

