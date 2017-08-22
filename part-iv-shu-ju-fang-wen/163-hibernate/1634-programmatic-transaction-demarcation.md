### 16.3.4编程式事务划分

开发者可以在应用程序的更高级别上对事务进行标定，而不用考虑低级别的数据访问执行了多少操作。这样不会对业务服务的实现进行限制；只需要定义一个Spring的`PlatformTransactionManager`即可。当然，`PlatformTransactionManager`可以从多处获取，但最好是通过`setTransactionManager(..)`方法以Bean来注入，正如`ProductDAO`应该由`setProductDao(..)`方法配置一样。下面的代码显示Spring应用程序上下文中的事务管理器和业务服务的定义，以及业务方法实现的示例：

```java
<beans>

    <bean id="myTxManager" class="org.springframework.orm.hibernate5.HibernateTransactionManager">
        <property name="sessionFactory" ref="mySessionFactory"/>
    </bean>

    <bean id="myProductService" class="product.ProductServiceImpl">
        <property name="transactionManager" ref="myTxManager"/>
        <property name="productDao" ref="myProductDao"/>
    </bean>

</beans>
public class ProductServiceImpl implements ProductService {

    private TransactionTemplate transactionTemplate;
    private ProductDao productDao;

    public void setTransactionManager(PlatformTransactionManager transactionManager) {
        this.transactionTemplate = new TransactionTemplate(transactionManager);
    }

    public void setProductDao(ProductDao productDao) {
        this.productDao = productDao;
    }

    public void increasePriceOfAllProductsInCategory(final String category) {
        this.transactionTemplate.execute(new TransactionCallbackWithoutResult() {
            public void doInTransactionWithoutResult(TransactionStatus status) {
                List productsToChange = this.productDao.loadProductsByCategory(category);
                // do the price increase...
            }
        });
    }
}
```

Spring的`TransactionInterceptor`允许任何检查的应用异常到`callback`代码中去，而`TransactionTemplate`还会非受检异常触发进行回调。`TransactionTemplate`则会因为非受检异常或者是由应用标记事务回滚\(通过`TransactionStatus`\)。`TransactionInterceptor`也是一样的处理逻辑，但是同时还允许基于方法配置回滚策略。

