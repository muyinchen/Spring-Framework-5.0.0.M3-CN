### 

### 13.5.2Example of declarative transaction implementation

Consider the following interface, and its attendant implementation. This example uses`Foo`and`Bar`classes as placeholders so that you can concentrate on the transaction usage without focusing on a particular domain model. For the purposes of this example, the fact that the`DefaultFooService`class throws`UnsupportedOperationException`instances in the body of each implemented method is good; it allows you to see transactions created and then rolled back in response to the`UnsupportedOperationException`instance.

```java
// the service interface that we want to make transactional

package x.y.service;

public interface FooService {

	Foo getFoo(String fooName);

	Foo getFoo(String fooName, String barName);

	void insertFoo(Foo foo);

	void updateFoo(Foo foo);

}

```



```java
// an implementation of the above interface

package x.y.service;

public class DefaultFooService implements FooService {

	public Foo getFoo(String fooName) {
		throw new UnsupportedOperationException();
	}

	public Foo getFoo(String fooName, String barName) {
		throw new UnsupportedOperationException();
	}

	public void insertFoo(Foo foo) {
		throw new UnsupportedOperationException();
	}

	public void updateFoo(Foo foo) {
		throw new UnsupportedOperationException();
	}

}
```

Assume that the first two methods of the`FooService`interface,`getFoo(String)`and`getFoo(String, String)`, must execute in the context of a transaction with read-only semantics, and that the other methods,`insertFoo(Foo)`and`updateFoo(Foo)`, must execute in the context of a transaction with read-write semantics. The following configuration is explained in detail in the next few paragraphs.

```java
<!-- from the file 'context.xml' -->
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

	<!-- this is the service object that we want to make transactional -->
	<bean id="fooService" class="x.y.service.DefaultFooService"/>

	<!-- the transactional advice (what 'happens'; see the <aop:advisor/> bean below) -->
	<tx:advice id="txAdvice" transaction-manager="txManager">
		<!-- the transactional semantics... -->
		<tx:attributes>
			<!-- all methods starting with 'get' are read-only -->
			<tx:method name="get*" read-only="true"/>
			<!-- other methods use the default transaction settings (see below) -->
			<tx:method name="*"/>
		</tx:attributes>
	</tx:advice>

	<!-- ensure that the above transactional advice runs for any execution
		of an operation defined by the FooService interface -->
	<aop:config>
		<aop:pointcut id="fooServiceOperation" expression="execution(* x.y.service.FooService.*(..))"/>
		<aop:advisor advice-ref="txAdvice" pointcut-ref="fooServiceOperation"/>
	</aop:config>

	<!-- don't forget the DataSource -->
	<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
		<property name="driverClassName" value="oracle.jdbc.driver.OracleDriver"/>
		<property name="url" value="jdbc:oracle:thin:@rj-t42:1521:elvis"/>
		<property name="username" value="scott"/>
		<property name="password" value="tiger"/>
	</bean>

	<!-- similarly, don't forget the PlatformTransactionManager -->
	<bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
		<property name="dataSource" ref="dataSource"/>
	</bean>

	<!-- other <bean/> definitions here -->

</beans>
```

Examine the preceding configuration. You want to make a service object, the`fooService`bean, transactional. The transaction semantics to apply are encapsulated in the`definition. The`definition reads as "_…​ all methods on starting with'get'are to execute in the context of a read-only transaction, and all other methods are to execute with the default transaction semantics_". The`transaction-manager`attribute of the```tag is set to the name of the``PlatformTransactionManager`bean that is going to*drive*the transactions, in this case, the`txManager\`bean.

| ![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/tip.png) |
| :--- |
| You can omit the`transaction-manager`attribute in the transactional advice \(```) if the bean name of the``PlatformTransactionManager`that you want to wire in has the name`transactionManager`. If the`PlatformTransactionManager`bean that you want to wire in has any other name, then you must use the`transaction-manager\`attribute explicitly, as in the preceding example. |

The`<aop:config/>`definition ensures that the transactional advice defined by the`txAdvice`bean executes at the appropriate points in the program. First you define a pointcut that matches the execution of any operation defined in the`FooService`interface \(`fooServiceOperation`\). Then you associate the pointcut with the`txAdvice`using an advisor. The result indicates that at the execution of a`fooServiceOperation`, the advice defined by`txAdvice`will be run.

The expression defined within the`<aop:pointcut/>`element is an AspectJ pointcut expression; see[Chapter7,Aspect Oriented Programming with Spring](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/aop.html)for more details on pointcut expressions in Spring.

A common requirement is to make an entire service layer transactional. The best way to do this is simply to change the pointcut expression to match any operation in your service layer. For example:

```java
<aop:config>
	<aop:pointcut id="fooServiceMethods" expression="execution(* x.y.service.*.*(..))"/>
	<aop:advisor advice-ref="txAdvice" pointcut-ref="fooServiceMethods"/>
</aop:config>
```

| ![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png "\[Note\]") |
| :--- |
| _In this example it is assumed that all your service interfaces are defined in the`x.y.service`package; see_[_Chapter7,Aspect Oriented Programming with Spring_](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/aop.html)_for more details._ |

Now that we’ve analyzed the configuration, you may be asking yourself, "_Okay…​ but what does all this configuration actually do?_".

The above configuration will be used to create a transactional proxy around the object that is created from the`fooService`bean definition. The proxy will be configured with the transactional advice, so that when an appropriate method is invoked_on the proxy_, a transaction is started, suspended, marked as read-only, and so on, depending on the transaction configuration associated with that method. Consider the following program that test drives the above configuration:

```java
public final class Boot {

	public static void main(final String[] args) throws Exception {
		ApplicationContext ctx = new ClassPathXmlApplicationContext("context.xml", Boot.class);
		FooService fooService = (FooService) ctx.getBean("fooService");
		fooService.insertFoo (new Foo());
	}
}
```

The output from running the preceding program will resemble the following. \(The Log4J output and the stack trace from the UnsupportedOperationException thrown by the insertFoo\(..\) method of the DefaultFooService class have been truncated for clarity.\)

```java
<!-- the Spring container is starting up... -->
[AspectJInvocationContextExposingAdvisorAutoProxyCreator] - Creating implicit proxy for bean 'fooService' with 0 common interceptors and 1 specific interceptors

<!-- the DefaultFooService is actually proxied -->
[JdkDynamicAopProxy] - Creating JDK dynamic proxy for [x.y.service.DefaultFooService]

<!-- ... the insertFoo(..) method is now being invoked on the proxy -->
[TransactionInterceptor] - Getting transaction for x.y.service.FooService.insertFoo

<!-- the transactional advice kicks in here... -->
[DataSourceTransactionManager] - Creating new transaction with name [x.y.service.FooService.insertFoo]
[DataSourceTransactionManager] - Acquired Connection [org.apache.commons.dbcp.PoolableConnection@a53de4] for JDBC transaction

<!-- the insertFoo(..) method from DefaultFooService throws an exception... -->
[RuleBasedTransactionAttribute] - Applying rules to determine whether transaction should rollback on java.lang.UnsupportedOperationException
[TransactionInterceptor] - Invoking rollback for transaction on x.y.service.FooService.insertFoo due to throwable [java.lang.UnsupportedOperationException]

<!-- and the transaction is rolled back (by default, RuntimeException instances cause rollback) -->
[DataSourceTransactionManager] - Rolling back JDBC transaction on Connection [org.apache.commons.dbcp.PoolableConnection@a53de4]
[DataSourceTransactionManager] - Releasing JDBC Connection after transaction
[DataSourceUtils] - Returning JDBC Connection to DataSource

Exception in thread "main" java.lang.UnsupportedOperationException at x.y.service.DefaultFooService.insertFoo(DefaultFooService.java:14)
<!-- AOP infrastructure stack trace elements removed for clarity -->
at $Proxy0.insertFoo(Unknown Source)
at Boot.main(Boot.java:11)
```





