### 16.3.4Programmatic transaction demarcation

You can demarcate transactions in a higher level of the application, on top of such lower-level data access services spanning any number of operations. Nor do restrictions exist on the implementation of the surrounding business service; it just needs a Spring`PlatformTransactionManager`. Again, the latter can come from anywhere, but preferably as a bean reference through a`setTransactionManager(..)`method, just as the`productDAO`should be set by a`setProductDao(..)`method. The following snippets show a transaction manager and a business service definition in a Spring application context, and an example for a business method implementation:

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

Spring’s`TransactionInterceptor`allows any checked application exception to be thrown with the callback code, while`TransactionTemplate`is restricted to unchecked exceptions within the callback.`TransactionTemplate`triggers a rollback in case of an unchecked application exception, or if the transaction is marked rollback-only by the application \(via`TransactionStatus`\).`TransactionInterceptor`behaves the same way by default but allows configurable rollback policies per method.

