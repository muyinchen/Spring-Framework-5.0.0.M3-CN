### 16.3.3Declarative transaction demarcation

We recommend that you use Spring’s declarative transaction support, which enables you to replace explicit transaction demarcation API calls in your Java code with an AOP transaction interceptor. This transaction interceptor can be configured in a Spring container using either Java annotations or XML. This declarative transaction capability allows you to keep business services free of repetitive transaction demarcation code and to focus on adding business logic, which is the real value of your application.

| ![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| :--- |
| Prior to continuing, you are_strongly_encouraged to read[Section13.5, “Declarative transaction management”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/transaction.html#transaction-declarative)if you have not done so. |

You may annotate the service layer with`@Transactional`annotations and instruct the Spring container to find these annotations and provide transactional semantics for these annotated methods.

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

All you need to set up in the container is the`PlatformTransactionManager`implementation as a bean as well as a "&lt;tx:annotation-driven/&gt;" entry, opting into`@Transactional`processing at runtime.

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







