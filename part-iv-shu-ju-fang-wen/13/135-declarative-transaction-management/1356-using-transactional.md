### 13.5.6Using @Transactional

In addition to the XML-based declarative approach to transaction configuration, you can use an annotation-based approach. Declaring transaction semantics directly in the Java source code puts the declarations much closer to the affected code. There is not much danger of undue coupling, because code that is meant to be used transactionally is almost always deployed that way anyway.

| ![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| :--- |
| The standard`javax.transaction.Transactional`annotation is also supported as a drop-in replacement to Spring’s own annotation. Please refer to JTA 1.2 documentation for more details. |

The ease-of-use afforded by the use of the`@Transactional`annotation is best illustrated with an example, which is explained in the text that follows. Consider the following class definition:

```java
// the service class that we want to make transactional
@Transactional
public class DefaultFooService implements FooService {

	Foo getFoo(String fooName);

	Foo getFoo(String fooName, String barName);

	void insertFoo(Foo foo);

	void updateFoo(Foo foo);
}
```

When the above POJO is defined as a bean in a Spring IoC container, the bean instance can be made transactional by adding merely_one_line of XML configuration:

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

	<!-- enable the configuration of transactional behavior based on annotations -->
	<tx:annotation-driven transaction-manager="txManager"/><!-- a PlatformTransactionManager is still required -->
	<bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
		<!-- (this dependency is defined somewhere else) -->
		<property name="dataSource" ref="dataSource"/>
	</bean>

	<!-- other <bean/> definitions here -->

</beans>
```

| ![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/tip.png.pagespeed.ce.w22Wv-tZ37.png) |
| :--- |
| You can omit the`transaction-manager`attribute in the```tag if the bean name of the``PlatformTransactionManager`that you want to wire in has the name`transactionManager`. If the`PlatformTransactionManager`bean that you want to dependency-inject has any other name, then you have to use the`transaction-manager\`attribute explicitly, as in the preceding example. |

| ![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| :--- |


| The`@EnableTransactionManagement`annotation provides equivalent support if you are using Java based configuration. Simply add the annotation to a`@Configuration`class. See the javadocs for full details. |
| :--- |


**Method visibility and @Transactional**

When using proxies, you should apply the`@Transactional`annotation only to methods with_public_visibility. If you do annotate protected, private or package-visible methods with the`@Transactional`annotation, no error is raised, but the annotated method does not exhibit the configured transactional settings. Consider the use of AspectJ \(see below\) if you need to annotate non-public methods.

You can place the`@Transactional`annotation before an interface definition, a method on an interface, a class definition, or a_public_method on a class. However, the mere presence of the`@Transactional`annotation is not enough to activate the transactional behavior. The`@Transactional`annotation is simply metadata that can be consumed by some runtime infrastructure that is`@Transactional`-aware and that can use the metadata to configure the appropriate beans with transactional behavior. In the preceding example, the\`\`element_switches on_the transactional behavior.

| ![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/tip.png.pagespeed.ce.w22Wv-tZ37.png) |
| :--- |
| Spring recommends that you only annotate concrete classes \(and methods of concrete classes\) with the`@Transactional`annotation, as opposed to annotating interfaces. You certainly can place the`@Transactional`annotation on an interface \(or an interface method\), but this works only as you would expect it to if you are using interface-based proxies. The fact that Java annotations are_not inherited from interfaces_means that if you are using class-based proxies \(`proxy-target-class="true"`\) or the weaving-based aspect \(`mode="aspectj"`\), then the transaction settings are not recognized by the proxying and weaving infrastructure, and the object will not be wrapped in a transactional proxy, which would be decidedly_bad_. |

| ![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| :--- |
| In proxy mode \(which is the default\), only external method calls coming in through the proxy are intercepted. This means that self-invocation, in effect, a method within the target object calling another method of the target object, will not lead to an actual transaction at runtime even if the invoked method is marked with`@Transactional`. Also, the proxy must be fully initialized to provide the expected behaviour so you should not rely on this feature in your initialization code, i.e.`@PostConstruct`. |

Consider the use of AspectJ mode \(see mode attribute in table below\) if you expect self-invocations to be wrapped with transactions as well. In this case, there will not be a proxy in the first place; instead, the target class will be weaved \(that is, its byte code will be modified\) in order to turn`@Transactional`into runtime behavior on any kind of method.

  
**Table13.2.Annotation driven transaction settings**

| XML Attribute | Annotation Attribute | Default | Description |
| :--- | :--- | :--- | :--- |
| `transaction-manager` | N/A \(See`TransactionManagementConfigurer`javadocs\) | transactionManager | Name of transaction manager to use. Only required if the name of the transaction manager is not`transactionManager`, as in the example above. |
| `mode` | `mode` | proxy | The default mode "proxy" processes annotated beans to be proxied using Spring’s AOP framework \(following proxy semantics, as discussed above, applying to method calls coming in through the proxy only\). The alternative mode "aspectj" instead weaves the affected classes with Spring’s AspectJ transaction aspect, modifying the target class byte code to apply to any kind of method call. AspectJ weaving requires spring-aspects.jar in the classpath as well as load-time weaving \(or compile-time weaving\) enabled. \(See[the section called “Spring configuration”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/aop.html#aop-aj-ltw-spring)for details on how to set up load-time weaving.\) |
| `proxy-target-class` | `proxyTargetClass` | false | Applies to proxy mode only. Controls what type of transactional proxies are created for classes annotated with the`@Transactional`annotation. If the`proxy-target-class`attribute is set to`true`, then class-based proxies are created. If`proxy-target-class`is`false`or if the attribute is omitted, then standard JDK interface-based proxies are created. \(See[Section7.6, “Proxying mechanisms”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/aop.html#aop-proxying)for a detailed examination of the different proxy types.\) |
| `order` | `order` | Ordered.LOWEST\_PRECEDENCE | Defines the order of the transaction advice that is applied to beans annotated with`@Transactional`. \(For more information about the rules related to ordering of AOP advice, see[the section called “Advice ordering”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/aop.html#aop-ataspectj-advice-ordering).\) No specified ordering means that the AOP subsystem determines the order of the advice. |

| ![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| :--- |
| The`proxy-target-class`attribute controls what type of transactional proxies are created for classes annotated with the`@Transactional`annotation. If`proxy-target-class`is set to`true`, class-based proxies are created. If`proxy-target-class`is`false`or if the attribute is omitted, standard JDK interface-based proxies are created. \(See[Section7.6, “Proxying mechanisms”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/aop.html#aop-proxying)for a discussion of the different proxy types.\) |

| ![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| :--- |
| `@EnableTransactionManagement`and```only looks for``@Transactional`on beans in the same application context they are defined in. This means that, if you put annotation driven configuration in a`WebApplicationContext`for a`DispatcherServlet`, it only checks for`@Transactional\`beans in your controllers, and not your services. See[Section18.2, “The DispatcherServlet”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-servlet)for more information. |

  
The most derived location takes precedence when evaluating the transactional settings for a method. In the case of the following example, the`DefaultFooService`class is annotated at the class level with the settings for a read-only transaction, but the`@Transactional`annotation on the`updateFoo(Foo)`method in the same class takes precedence over the transactional settings defined at the class level.

```java
@Transactional(readOnly = true)
public class DefaultFooService implements FooService {

	public Foo getFoo(String fooName) {
		// do something
	}

	// these settings have precedence for this method
	@Transactional(readOnly = false, propagation = Propagation.REQUIRES_NEW)
	public void updateFoo(Foo foo) {
		// do something
	}
}
```

#### @Transactional settings

The`@Transactional`annotation is metadata that specifies that an interface, class, or method must have transactional semantics; for example, "_start a brand new read-only transaction when this method is invoked, suspending any existing transaction_". The default`@Transactional`settings are as follows:

* Propagation setting is`PROPAGATION_REQUIRED.`

* Isolation level is`ISOLATION_DEFAULT.`

* Transaction is read/write.

* Transaction timeout defaults to the default timeout of the underlying transaction system, or to none if timeouts are not supported.

* Any`RuntimeException`triggers rollback, and any checked`Exception`does not.

These default settings can be changed; the various properties of the`@Transactional`annotation are summarized in the following table:

**Table13.3.@Transactional Settings**

| Property | Type | Description |
| :--- | :--- | :--- |
| [value](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/transaction.html#tx-multiple-tx-mgrs-with-attransactional) | String | Optional qualifier specifying the transaction manager to be used. |
| [propagation](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/transaction.html#tx-propagation) | enum:`Propagation` | Optional propagation setting. |
| `isolation` | enum:`Isolation` | Optional isolation level. |
| `readOnly` | boolean | Read/write vs. read-only transaction |
| `timeout` | int \(in seconds granularity\) | Transaction timeout. |
| `rollbackFor` | Array of`Class`objects, which must be derived from`Throwable.` | Optional array of exception classes that_must_cause rollback. |
| `rollbackForClassName` | Array of class names. Classes must be derived from`Throwable.` | Optional array of names of exception classes that_must_cause rollback. |
| `noRollbackFor` | Array of`Class`objects, which must be derived from`Throwable.` | Optional array of exception classes that_must not_cause rollback. |
| `noRollbackForClassName` | Array of`String`class names, which must be derived from`Throwable.` | Optional array of names of exception classes that_must not_cause rollback. |

Currently you cannot have explicit control over the name of a transaction, where 'name' means the transaction name that will be shown in a transaction monitor, if applicable \(for example, WebLogic’s transaction monitor\), and in logging output. For declarative transactions, the transaction name is always the fully-qualified class name + "." + method name of the transactionally-advised class. For example, if the`handlePayment(..)`method of the`BusinessService`class started a transaction, the name of the transaction would be:`com.foo.BusinessService.handlePayment`.

#### Multiple Transaction Managers with @Transactional

Most Spring applications only need a single transaction manager, but there may be situations where you want multiple independent transaction managers in a single application. The value attribute of the`@Transactional`annotation can be used to optionally specify the identity of the`PlatformTransactionManager`to be used. This can either be the bean name or the qualifier value of the transaction manager bean. For example, using the qualifier notation, the following Java code

```java
public class TransactionalService {

	@Transactional("order")
	public void setSomething(String name) { ... }

	@Transactional("account")
	public void doSomething() { ... }
}
```

could be combined with the following transaction manager bean declarations in the application context.

```java
<tx:annotation-driven/>

	<bean id="transactionManager1" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
		...
		<qualifier value="order"/>
	</bean>

	<bean id="transactionManager2" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
		...
		<qualifier value="account"/>
	</bean>
```

In this case, the two methods on`TransactionalService`will run under separate transaction managers, differentiated by the "order" and "account" qualifiers. The default```target bean name``transactionManager\`will still be used if no specifically qualified PlatformTransactionManager bean is found.

#### Custom shortcut annotations

If you find you are repeatedly using the same attributes with`@Transactional`on many different methods, then[Spring’s meta-annotation support](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/beans.html#beans-meta-annotations)allows you to define custom shortcut annotations for your specific use cases. For example, defining the following annotations

```java
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Transactional("order")
public @interface OrderTx {
}

@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Transactional("account")
public @interface AccountTx {
}
```

allows us to write the example from the previous section as

```java
public class TransactionalService {

	@OrderTx
	public void setSomething(String name) { ... }

	@AccountTx
	public void doSomething() { ... }
}
```

Here we have used the syntax to define the transaction manager qualifier, but could also have included propagation behavior, rollback rules, timeouts etc.



