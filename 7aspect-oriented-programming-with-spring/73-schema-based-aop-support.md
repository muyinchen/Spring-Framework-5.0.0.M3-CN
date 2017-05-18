## 7.3 Schema-based AOP support

If you prefer an XML-based format, then Spring also offers support for defining aspects using the new "aop" namespace tags. The exact same pointcut expressions and advice kinds are supported as when using the @AspectJ style, hence in this section we will focus on the new *syntax* and refer the reader to the discussion in the previous section ([Section 7.2, “@AspectJ support”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#aop-ataspectj)) for an understanding of writing pointcut expressions and the binding of advice parameters.

To use the aop namespace tags described in this section, you need to import the `spring-aop` schema as described in [Chapter 38, *XML Schema-based configuration*](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#xsd-configuration). See [Section 38.2.7, “the aop schema”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#xsd-config-body-schemas-aop) for how to import the tags in the `aop` namespace.

Within your Spring configurations, all aspect and advisor elements must be placed within an `<aop:config>` element (you can have more than one `<aop:config>`element in an application context configuration). An `<aop:config>` element can contain pointcut, advisor, and aspect elements (note these must be declared in that order).

| ![[Warning]](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/images/warning.png.pagespeed.ce.aG27Rp2bJY.png) |
| ---------------------------------------- |
| The `<aop:config>` style of configuration makes heavy use of Spring’s [auto-proxying](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#aop-autoproxy) mechanism. This can cause issues (such as advice not being woven) if you are already using explicit auto-proxying via the use of `BeanNameAutoProxyCreator` or suchlike. The recommended usage pattern is to use either just the `<aop:config>` style, or just the `AutoProxyCreator` style. |

### 7.3.1 Declaring an aspect

Using the schema support, an aspect is simply a regular Java object defined as a bean in your Spring application context. The state and behavior is captured in the fields and methods of the object, and the pointcut and advice information is captured in the XML.

An aspect is declared using the <aop:aspect> element, and the backing bean is referenced using the `ref` attribute:

```xml
<aop:config>
	<aop:aspect id="myAspect" ref="aBean">
		...
	</aop:aspect>
</aop:config>

<bean id="aBean" class="...">
	...
</bean>
```

The bean backing the aspect (" `aBean`" in this case) can of course be configured and dependency injected just like any other Spring bean.

### 7.3.2 Declaring a pointcut

A named pointcut can be declared inside an <aop:config> element, enabling the pointcut definition to be shared across several aspects and advisors.

A pointcut representing the execution of any business service in the service layer could be defined as follows:

```xml
<aop:config>

	<aop:pointcut id="businessService"
		expression="execution(* com.xyz.myapp.service.*.*(..))"/>

</aop:config>
```

Note that the pointcut expression itself is using the same AspectJ pointcut expression language as described in [Section 7.2, “@AspectJ support”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#aop-ataspectj). If you are using the schema based declaration style, you can refer to named pointcuts defined in types (@Aspects) within the pointcut expression. Another way of defining the above pointcut would be:

```xml
<aop:config>

	<aop:pointcut id="businessService"
		expression="com.xyz.myapp.SystemArchitecture.businessService()"/>

</aop:config>
```

Assuming you have a `SystemArchitecture` aspect as described in [the section called “Sharing common pointcut definitions”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#aop-common-pointcuts).

Declaring a pointcut inside an aspect is very similar to declaring a top-level pointcut:

```xml
<aop:config>

	<aop:aspect id="myAspect" ref="aBean">

		<aop:pointcut id="businessService"
			expression="execution(* com.xyz.myapp.service.*.*(..))"/>

		...

	</aop:aspect>

</aop:config>
```

Much the same way in an @AspectJ aspect, pointcuts declared using the schema based definition style may collect join point context. For example, the following pointcut collects the 'this' object as the join point context and passes it to advice:

```xml
<aop:config>

	<aop:aspect id="myAspect" ref="aBean">

		<aop:pointcut id="businessService"
			expression="execution(* com.xyz.myapp.service.*.*(..)) &amp;&amp; this(service)"/>

		<aop:before pointcut-ref="businessService" method="monitor"/>

		...

	</aop:aspect>

</aop:config>
```

The advice must be declared to receive the collected join point context by including parameters of the matching names:

```java
public void monitor(Object service) {
	...
}
```

When combining pointcut sub-expressions, '&&' is awkward within an XML document, and so the keywords 'and', 'or' and 'not' can be used in place of '&&', '||' and '!' respectively. For example, the previous pointcut may be better written as:

```xml
<aop:config>

	<aop:aspect id="myAspect" ref="aBean">

		<aop:pointcut id="businessService"
			expression="execution(* com.xyz.myapp.service.*.*(..)) **and** this(service)"/>

		<aop:before pointcut-ref="businessService" method="monitor"/>

		...
	</aop:aspect>
</aop:config>
```

Note that pointcuts defined in this way are referred to by their XML id and cannot be used as named pointcuts to form composite pointcuts. The named pointcut support in the schema based definition style is thus more limited than that offered by the @AspectJ style.

### 7.3.3 Declaring advice

The same five advice kinds are supported as for the @AspectJ style, and they have exactly the same semantics.

#### Before advice

Before advice runs before a matched method execution. It is declared inside an `<aop:aspect>` using the <aop:before> element.

```xml
<aop:aspect id="beforeExample" ref="aBean">

	<aop:before
		pointcut-ref="dataAccessOperation"
		method="doAccessCheck"/>

	...

</aop:aspect>
```

Here `dataAccessOperation` is the id of a pointcut defined at the top ( `<aop:config>`) level. To define the pointcut inline instead, replace the `pointcut-ref` attribute with a `pointcut` attribute:

```xml
<aop:aspect id="beforeExample" ref="aBean">

	<aop:before
		pointcut="execution(* com.xyz.myapp.dao.*.*(..))"
		method="doAccessCheck"/>

	...

</aop:aspect>
```

As we noted in the discussion of the @AspectJ style, using named pointcuts can significantly improve the readability of your code.

The method attribute identifies a method ( `doAccessCheck`) that provides the body of the advice. This method must be defined for the bean referenced by the aspect element containing the advice. Before a data access operation is executed (a method execution join point matched by the pointcut expression), the "doAccessCheck" method on the aspect bean will be invoked.

#### After returning advice

After returning advice runs when a matched method execution completes normally. It is declared inside an `<aop:aspect>` in the same way as before advice. For example:

```xml
<aop:aspect id="afterReturningExample" ref="aBean">

	<aop:after-returning
		pointcut-ref="dataAccessOperation"
		method="doAccessCheck"/>

	...

</aop:aspect>
```

Just as in the @AspectJ style, it is possible to get hold of the return value within the advice body. Use the returning attribute to specify the name of the parameter to which the return value should be passed:

```xml
<aop:aspect id="afterReturningExample" ref="aBean">

	<aop:after-returning
		pointcut-ref="dataAccessOperation"
		returning="retVal"
		method="doAccessCheck"/>

	...

</aop:aspect>
```

The doAccessCheck method must declare a parameter named `retVal`. The type of this parameter constrains matching in the same way as described for @AfterReturning. For example, the method signature may be declared as:

```java
public void doAccessCheck(Object retVal) {...
```

#### After throwing advice

After throwing advice executes when a matched method execution exits by throwing an exception. It is declared inside an `<aop:aspect>` using the after-throwing element:

```xml
<aop:aspect id="afterThrowingExample" ref="aBean">

	<aop:after-throwing
		pointcut-ref="dataAccessOperation"
		method="doRecoveryActions"/>

	...

</aop:aspect>
```

Just as in the @AspectJ style, it is possible to get hold of the thrown exception within the advice body. Use the throwing attribute to specify the name of the parameter to which the exception should be passed:

```xml
<aop:aspect id="afterThrowingExample" ref="aBean">

	<aop:after-throwing
		pointcut-ref="dataAccessOperation"
		throwing="dataAccessEx"
		method="doRecoveryActions"/>

	...

</aop:aspect>
```

The doRecoveryActions method must declare a parameter named `dataAccessEx`. The type of this parameter constrains matching in the same way as described for @AfterThrowing. For example, the method signature may be declared as:

```java
public void doRecoveryActions(DataAccessException dataAccessEx) {...
```

#### After (finally) advice

After (finally) advice runs however a matched method execution exits. It is declared using the `after` element:

```xml
<aop:aspect id="afterFinallyExample" ref="aBean">

	<aop:after
		pointcut-ref="dataAccessOperation"
		method="doReleaseLock"/>

	...

</aop:aspect>
```

#### Around advice

The final kind of advice is around advice. Around advice runs "around" a matched method execution. It has the opportunity to do work both before and after the method executes, and to determine when, how, and even if, the method actually gets to execute at all. Around advice is often used if you need to share state before and after a method execution in a thread-safe manner (starting and stopping a timer for example). Always use the least powerful form of advice that meets your requirements; don’t use around advice if simple before advice would do.

Around advice is declared using the `aop:around` element. The first parameter of the advice method must be of type `ProceedingJoinPoint`. Within the body of the advice, calling `proceed()` on the `ProceedingJoinPoint` causes the underlying method to execute. The `proceed` method may also be calling passing in an `Object[]` - the values in the array will be used as the arguments to the method execution when it proceeds. See [the section called “Around advice”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#aop-ataspectj-around-advice) for notes on calling proceed with an `Object[]`.

```xml
<aop:aspect id="aroundExample" ref="aBean">

	<aop:around
		pointcut-ref="businessService"
		method="doBasicProfiling"/>

	...

</aop:aspect>
```

The implementation of the `doBasicProfiling` advice would be exactly the same as in the @AspectJ example (minus the annotation of course):

```java
public Object doBasicProfiling(ProceedingJoinPoint pjp) throws Throwable {
	// start stopwatch
	Object retVal = pjp.proceed();
	// stop stopwatch
	return retVal;
}
```

#### Advice parameters

The schema based declaration style supports fully typed advice in the same way as described for the @AspectJ support - by matching pointcut parameters by name against advice method parameters. See [the section called “Advice parameters”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#aop-ataspectj-advice-params) for details. If you wish to explicitly specify argument names for the advice methods (not relying on the detection strategies previously described) then this is done using the `arg-names` attribute of the advice element, which is treated in the same manner to the "argNames" attribute in an advice annotation as described in [the section called “Determining argument names”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#aop-ataspectj-advice-params-names). For example:

```xml
<aop:before
	pointcut="com.xyz.lib.Pointcuts.anyPublicMethod() and @annotation(auditable)"
	method="audit"
	arg-names="auditable"/>
```

The `arg-names` attribute accepts a comma-delimited list of parameter names.

Find below a slightly more involved example of the XSD-based approach that illustrates some around advice used in conjunction with a number of strongly typed parameters.

```java
package x.y.service;

public interface FooService {

	Foo getFoo(String fooName, int age);
}

public class DefaultFooService implements FooService {

	public Foo getFoo(String name, int age) {
		return new Foo(name, age);
	}
}
```

Next up is the aspect. Notice the fact that the `profile(..)` method accepts a number of strongly-typed parameters, the first of which happens to be the join point used to proceed with the method call: the presence of this parameter is an indication that the `profile(..)` is to be used as `around` advice:

```java
package x.y;

import org.aspectj.lang.ProceedingJoinPoint;
import org.springframework.util.StopWatch;

public class SimpleProfiler {

	public Object profile(ProceedingJoinPoint call, String name, int age) throws Throwable {
		StopWatch clock = new StopWatch("Profiling for '" + name + "' and '" + age + "'");
		try {
			clock.start(call.toShortString());
			return call.proceed();
		} finally {
			clock.stop();
			System.out.println(clock.prettyPrint());
		}
	}
}
```

Finally, here is the XML configuration that is required to effect the execution of the above advice for a particular join point:

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:aop="http://www.springframework.org/schema/aop"
	xsi:schemaLocation="
		http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">

	<!-- this is the object that will be proxied by Spring's AOP infrastructure -->
	<bean id="fooService" class="x.y.service.DefaultFooService"/>

	<!-- this is the actual advice itself -->
	<bean id="profiler" class="x.y.SimpleProfiler"/>

	<aop:config>
		<aop:aspect ref="profiler">

			<aop:pointcut id="theExecutionOfSomeFooServiceMethod"
				expression="execution(* x.y.service.FooService.getFoo(String,int))
				and args(name, age)"/>

			<aop:around pointcut-ref="theExecutionOfSomeFooServiceMethod"
				method="profile"/>

		</aop:aspect>
	</aop:config>

</beans>
```

If we had the following driver script, we would get output something like this on standard output:

```java
import org.springframework.beans.factory.BeanFactory;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import x.y.service.FooService;

public final class Boot {

	public static void main(final String[] args) throws Exception {
		BeanFactory ctx = new ClassPathXmlApplicationContext("x/y/plain.xml");
		FooService foo = (FooService) ctx.getBean("fooService");
		foo.getFoo("Pengo", 12);
	}
}
```

```java
StopWatch 'Profiling for 'Pengo' and '12'': running time (millis) = 0
-----------------------------------------
ms     %     Task name
-----------------------------------------
00000  ?  execution(getFoo)
```

#### Advice ordering

When multiple advice needs to execute at the same join point (executing method) the ordering rules are as described in [the section called “Advice ordering”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#aop-ataspectj-advice-ordering). The precedence between aspects is determined by either adding the `Order` annotation to the bean backing the aspect or by having the bean implement the `Ordered`interface.

### 7.3.4 Introductions

Introductions (known as inter-type declarations in AspectJ) enable an aspect to declare that advised objects implement a given interface, and to provide an implementation of that interface on behalf of those objects.

An introduction is made using the `aop:declare-parents` element inside an `aop:aspect` This element is used to declare that matching types have a new parent (hence the name). For example, given an interface `UsageTracked`, and an implementation of that interface `DefaultUsageTracked`, the following aspect declares that all implementors of service interfaces also implement the `UsageTracked` interface. (In order to expose statistics via JMX for example.)

```xml
<aop:aspect id="usageTrackerAspect" ref="usageTracking">

	<aop:declare-parents
		types-matching="com.xzy.myapp.service.*+"
		implement-interface="com.xyz.myapp.service.tracking.UsageTracked"
		default-impl="com.xyz.myapp.service.tracking.DefaultUsageTracked"/>

	<aop:before
		pointcut="com.xyz.myapp.SystemArchitecture.businessService()
			and this(usageTracked)"
			method="recordUsage"/>

</aop:aspect>
```

The class backing the `usageTracking` bean would contain the method:

```java
public void recordUsage(UsageTracked usageTracked) {
	usageTracked.incrementUseCount();
}
```

The interface to be implemented is determined by `implement-interface` attribute. The value of the `types-matching` attribute is an AspectJ type pattern :- any bean of a matching type will implement the `UsageTracked` interface. Note that in the before advice of the above example, service beans can be directly used as implementations of the `UsageTracked` interface. If accessing a bean programmatically you would write the following:

```java
UsageTracked usageTracked = (UsageTracked) context.getBean("myService");
```

### 7.3.5 Aspect instantiation models

The only supported instantiation model for schema-defined aspects is the singleton model. Other instantiation models may be supported in future releases.

### 7.3.6 Advisors

The concept of "advisors" is brought forward from the AOP support defined in Spring 1.2 and does not have a direct equivalent in AspectJ. An advisor is like a small self-contained aspect that has a single piece of advice. The advice itself is represented by a bean, and must implement one of the advice interfaces described in[Section 8.3.2, “Advice types in Spring”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#aop-api-advice-types). Advisors can take advantage of AspectJ pointcut expressions though.

Spring supports the advisor concept with the `<aop:advisor>` element. You will most commonly see it used in conjunction with transactional advice, which also has its own namespace support in Spring. Here’s how it looks:

```xml
<aop:config>

	<aop:pointcut id="businessService"
		expression="execution(* com.xyz.myapp.service.*.*(..))"/>

	<aop:advisor
		pointcut-ref="businessService"
		advice-ref="tx-advice"/>

</aop:config>

<tx:advice id="tx-advice">
	<tx:attributes>
		<tx:method name="*" propagation="REQUIRED"/>
	</tx:attributes>
</tx:advice>
```

As well as the `pointcut-ref` attribute used in the above example, you can also use the `pointcut` attribute to define a pointcut expression inline.

To define the precedence of an advisor so that the advice can participate in ordering, use the `order` attribute to define the `Ordered` value of the advisor.

### 7.3.7 Example

Let’s see how the concurrent locking failure retry example from [Section 7.2.7, “Example”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#aop-ataspectj-example) looks when rewritten using the schema support.

The execution of business services can sometimes fail due to concurrency issues (for example, deadlock loser). If the operation is retried, it is quite likely it will succeed next time round. For business services where it is appropriate to retry in such conditions (idempotent operations that don’t need to go back to the user for conflict resolution), we’d like to transparently retry the operation to avoid the client seeing a `PessimisticLockingFailureException`. This is a requirement that clearly cuts across multiple services in the service layer, and hence is ideal for implementing via an aspect.

Because we want to retry the operation, we’ll need to use around advice so that we can call proceed multiple times. Here’s how the basic aspect implementation looks (it’s just a regular Java class using the schema support):

```java
public class ConcurrentOperationExecutor implements Ordered {

	private static final int DEFAULT_MAX_RETRIES = 2;

	private int maxRetries = DEFAULT_MAX_RETRIES;
	private int order = 1;

	public void setMaxRetries(int maxRetries) {
		this.maxRetries = maxRetries;
	}

	public int getOrder() {
		return this.order;
	}

	public void setOrder(int order) {
		this.order = order;
	}

	public Object doConcurrentOperation(ProceedingJoinPoint pjp) throws Throwable {
		int numAttempts = 0;
		PessimisticLockingFailureException lockFailureException;
		do {
			numAttempts++;
			try {
				return pjp.proceed();
			}
			catch(PessimisticLockingFailureException ex) {
				lockFailureException = ex;
			}
		} while(numAttempts <= this.maxRetries);
		throw lockFailureException;
	}

}
```

Note that the aspect implements the `Ordered` interface so we can set the precedence of the aspect higher than the transaction advice (we want a fresh transaction each time we retry). The `maxRetries` and `order` properties will both be configured by Spring. The main action happens in the `doConcurrentOperation` around advice method. We try to proceed, and if we fail with a `PessimisticLockingFailureException` we simply try again unless we have exhausted all of our retry attempts.

| ![[Note]](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| ---------------------------------------- |
| This class is identical to the one used in the @AspectJ example, but with the annotations removed. |

The corresponding Spring configuration is:

```xml
<aop:config>

	<aop:aspect id="concurrentOperationRetry" ref="concurrentOperationExecutor">

		<aop:pointcut id="idempotentOperation"
			expression="execution(* com.xyz.myapp.service.*.*(..))"/>

		<aop:around
			pointcut-ref="idempotentOperation"
			method="doConcurrentOperation"/>

	</aop:aspect>

</aop:config>

<bean id="concurrentOperationExecutor"
	class="com.xyz.myapp.service.impl.ConcurrentOperationExecutor">
		<property name="maxRetries" value="3"/>
		<property name="order" value="100"/>
</bean>
```

Notice that for the time being we assume that all business services are idempotent. If this is not the case we can refine the aspect so that it only retries genuinely idempotent operations, by introducing an `Idempotent` annotation:

```java
@Retention(RetentionPolicy.RUNTIME)
public @interface Idempotent {
	// marker annotation
}
```

and using the annotation to annotate the implementation of service operations. The change to the aspect to retry only idempotent operations simply involves refining the pointcut expression so that only `@Idempotent` operations match:

```java
<aop:pointcut id="idempotentOperation"
		expression="execution(* com.xyz.myapp.service.*.*(..)) and
		@annotation(com.xyz.myapp.service.Idempotent)"/>
```