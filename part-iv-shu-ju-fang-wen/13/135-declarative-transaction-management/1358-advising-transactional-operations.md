### 13.5.8Advising transactional operations

Suppose you want to execute_both_transactional_and_some basic profiling advice. How do you effect this in the context of\`\`?

When you invoke the`updateFoo(Foo)`method, you want to see the following actions:

* Configured profiling aspect starts up.

* Transactional advice executes.

* Method on the advised object executes.

* Transaction commits.

* Profiling aspect reports exact duration of the whole transactional method invocation.

| ![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| :--- |
| This chapter is not concerned with explaining AOP in any great detail \(except as it applies to transactions\). See[Chapter7,_Aspect Oriented Programming with Spring_](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/aop.html)for detailed coverage of the following AOP configuration and AOP in general. |

Here is the code for a simple profiling aspect discussed above. The ordering of advice is controlled through the`Ordered`interface. For full details on advice ordering, see[the section called “Advice ordering”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/aop.html#aop-ataspectj-advice-ordering). .

```java
package x.y;

import org.aspectj.lang.ProceedingJoinPoint;
import org.springframework.util.StopWatch;
import org.springframework.core.Ordered;

public class SimpleProfiler implements Ordered {

	private int order;

	// allows us to control the ordering of advice
	public int getOrder() {
		return this.order;
	}

	public void setOrder(int order) {
		this.order = order;
	}

	// this method is the around advice
	public Object profile(ProceedingJoinPoint call) throws Throwable {
		Object returnValue;
		StopWatch clock = new StopWatch(getClass().getName());
		try {
			clock.start(call.toShortString());
			returnValue = call.proceed();
		} finally {
			clock.stop();
			System.out.println(clock.prettyPrint());
		}
		return returnValue;
	}
}
```



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

	<bean id="fooService" class="x.y.service.DefaultFooService"/>

	<!-- this is the aspect -->
	<bean id="profiler" class="x.y.SimpleProfiler">
		<!-- execute before the transactional advice (hence the lower order number) -->
		<property name="order" __value="1"__/>
	</bean>

	<tx:annotation-driven transaction-manager="txManager" __order="200"__/>

	<aop:config>
			<!-- this advice will execute around the transactional advice -->
			<aop:aspect id="profilingAspect" ref="profiler">
				<aop:pointcut id="serviceMethodWithReturnValue"
						expression="execution(!void x.y..*Service.*(..))"/>
				<aop:around method="profile" pointcut-ref="serviceMethodWithReturnValue"/>
			</aop:aspect>
	</aop:config>

	<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
		<property name="driverClassName" value="oracle.jdbc.driver.OracleDriver"/>
		<property name="url" value="jdbc:oracle:thin:@rj-t42:1521:elvis"/>
		<property name="username" value="scott"/>
		<property name="password" value="tiger"/>
	</bean>

	<bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
		<property name="dataSource" ref="dataSource"/>
	</bean>

</beans>
```

The result of the above configuration is a`fooService`bean that has profiling and transactional aspects applied to it_in the desired order_. You configure any number of additional aspects in similar fashion.

The following example effects the same setup as above, but uses the purely XML declarative approach.

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

	<bean id="fooService" class="x.y.service.DefaultFooService"/>

	<!-- the profiling advice -->
	<bean id="profiler" class="x.y.SimpleProfiler">
		<!-- execute before the transactional advice (hence the lower order number) -->
		__<property name="order" value="1__"/>
	</bean>

	<aop:config>
		<aop:pointcut id="entryPointMethod" expression="execution(* x.y..*Service.*(..))"/>
		<!-- will execute after the profiling advice (c.f. the order attribute) -->

		<aop:advisor advice-ref="txAdvice" pointcut-ref="entryPointMethod" __order="2__"/>
		<!-- order value is higher than the profiling aspect -->

		<aop:aspect id="profilingAspect" ref="profiler">
			<aop:pointcut id="serviceMethodWithReturnValue"
					expression="execution(!void x.y..*Service.*(..))"/>
			<aop:around method="profile" pointcut-ref="serviceMethodWithReturnValue"/>
		</aop:aspect>

	</aop:config>

	<tx:advice id="txAdvice" transaction-manager="txManager">
		<tx:attributes>
			<tx:method name="get*" read-only="true"/>
			<tx:method name="*"/>
		</tx:attributes>
	</tx:advice>

	<!-- other <bean/> definitions such as a DataSource and a PlatformTransactionManager here -->

</beans>
```

The result of the above configuration will be a`fooService`bean that has profiling and transactional aspects applied to it_in that order_. If you want the profiling advice to execute_after_the transactional advice on the way in, and_before_the transactional advice on the way out, then you simply swap the value of the profiling aspect bean’s`order`property so that it is higher than the transactional advice’s order value.

You configure additional aspects in similar fashion.

