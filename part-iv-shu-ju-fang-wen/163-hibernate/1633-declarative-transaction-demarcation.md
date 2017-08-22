### 16.3.3声明式事务划分

Spring团队建议开发者使用Spring声明式的事务支持，这样可以通过AOP事务拦截器来替代事务API的显式调用。AOP事务拦截器可以在Spring容器中使用XML或者Java的注解来进行配置。这种事务拦截器可以令开发者的代码和重复的事务代码相解耦，而开发者可以将精力更多集中在业务逻辑上，而业务逻辑才是应用的核心。

| ![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| :--- |
| 在继续之前，强烈建议开发者先查阅章节[Section13.5, 声明式事务管理](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/transaction.html#transaction-declarative)的内容。 |

开发者可以在服务层的代码使用注解`@Transactional`，这样可以让Spring容器找到这些注解，以对其中注解了的方法提供事务语义。

```java
public class ProductServiceImpl implements ProductService {

    private ProductDao productDao;

    public void setProductDao(ProductDao productDao) {
        this.productDao = productDao;
    }

    @Transactional
    public void increasePriceOfAllProductsInCategory(final String category) {
        List productsToChange = this.productDao.loadProductsByCategory(category);
        // ...
    }

    @Transactional(readOnly = true)
    public List<Product> findAllProducts() {
        return this.productDao.findAllProducts();
    }

}
```

开发者所需要做的就是在容器中配置`PlatformTransactionManager`的实现，或者是在XML中配置`<tx:annotation-driver/>`标签，这样就可以在运行时支持`@Transactional`的处理了。参考如下XML代码：

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:tx="http://www.springframework.org/schema/tx"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/tx
        http://www.springframework.org/schema/tx/spring-tx.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd">

    <!-- SessionFactory, DataSource, etc. omitted -->

    <bean id="transactionManager"
            class="org.springframework.orm.hibernate5.HibernateTransactionManager">
        <property name="sessionFactory" ref="sessionFactory"/>
    </bean>

    <tx:annotation-driven/>

    <bean id="myProductService" class="product.SimpleProductService">
        <property name="productDao" ref="myProductDao"/>
    </bean>

</beans>
```



