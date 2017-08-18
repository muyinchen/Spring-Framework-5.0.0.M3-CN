### 16.2.2异常转义

当在DAO层中使用Hibernate或者JPA的时候，开发者必须决定该如何处理持久化技术的一些原生异常。DAO层会根据选择技术的不同而抛出`HibernateException`或者`PersistenceException`。这些异常都属于运行时异常，所以无需显式声明和捕捉。同时，开发者同时还需要处理`IllegalArgumentException`和`IllegalStateException`这类异常。一般情况下，调用方通常只能将这一类异常视为致命的异常，除非他们想要自己的应用依赖于持久性技术原生的异常体系。如果需要捕获一些特定的错误，比如乐观锁获取失败一类的错误，只能选择调用方和实现策略耦合到一起。对于那些只基于某种特定ORM技术或者不需要特殊异常处理的应用来说，使用ORM本身的异常体系的代价是可以接受的。但是，Spring可以通过`@Repository`注解透明地应用异常转换，以解耦调用方和ORM技术的耦合：

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

上面的后置处理器`PersistenceExceptionTranslationPostProcessor`，会自动查找所有的异常转义器（实现`PersistenceExceptionTranslator`接口的Bean），并且拦截所有标记为`@Repository`注解的Bean，通过代理来拦截异常，然后通过`PersistenceExceptionTranslator`将DAO层异常转义后的异常抛出。

总而言之：开发者可以既基于简单的持久化技术的API和注解来实现DAO，同时还受益于Spring管理的事务，依赖注入和透明异常转换（如果需要）到Spring的自定义异常层次结构。

