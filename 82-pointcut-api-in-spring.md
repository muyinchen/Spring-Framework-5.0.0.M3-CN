## 8.2 Pointcut API in Spring

Let’s look at how Spring handles the crucial pointcut concept.

### 8.2.1 Concepts

Spring’s pointcut model enables pointcut reuse independent of advice types. It’s possible to target different advice using the same pointcut.

The `org.springframework.aop.Pointcut` interface is the central interface, used to target advices to particular classes and methods. The complete interface is shown below:

```java
public interface Pointcut {

	ClassFilter getClassFilter();

	MethodMatcher getMethodMatcher();

}
```

Splitting the `Pointcut` interface into two parts allows reuse of class and method matching parts, and fine-grained composition operations (such as performing a "union" with another method matcher).

The `ClassFilter` interface is used to restrict the pointcut to a given set of target classes. If the `matches()` method always returns true, all target classes will be matched:

```java
public interface ClassFilter {

	boolean matches(Class clazz);
}
```

The `MethodMatcher` interface is normally more important. The complete interface is shown below:

```java
public interface MethodMatcher {

	boolean matches(Method m, Class targetClass);

	boolean isRuntime();

	boolean matches(Method m, Class targetClass, Object[] args);
}
```

The `matches(Method, Class)` method is used to test whether this pointcut will ever match a given method on a target class. This evaluation can be performed when an AOP proxy is created, to avoid the need for a test on every method invocation. If the 2-argument matches method returns true for a given method, and the `isRuntime()` method for the MethodMatcher returns true, the 3-argument matches method will be invoked on every method invocation. This enables a pointcut to look at the arguments passed to the method invocation immediately before the target advice is to execute.

Most MethodMatchers are static, meaning that their `isRuntime()` method returns false. In this case, the 3-argument matches method will never be invoked.

| ![[Tip]](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/images/tip.png.pagespeed.ce.w22Wv-tZ37.png) |
| ---------------------------------------- |
| If possible, try to make pointcuts static, allowing the AOP framework to cache the results of pointcut evaluation when an AOP proxy is created. |

### 8.2.2 Operations on pointcuts

Spring supports operations on pointcuts: notably, *union* and *intersection*.

- Union means the methods that either pointcut matches.
- Intersection means the methods that both pointcuts match.
- Union is usually more useful.
- Pointcuts can be composed using the static methods in the *org.springframework.aop.support.Pointcuts* class, or using the *ComposablePointcut* class in the same package. However, using AspectJ pointcut expressions is usually a simpler approach.

### 8.2.3 AspectJ expression pointcuts

Since 2.0, the most important type of pointcut used by Spring is `org.springframework.aop.aspectj.AspectJExpressionPointcut`. This is a pointcut that uses an AspectJ supplied library to parse an AspectJ pointcut expression string.

See the previous chapter for a discussion of supported AspectJ pointcut primitives.

### 8.2.4 Convenience pointcut implementations

Spring provides several convenient pointcut implementations. Some can be used out of the box; others are intended to be subclassed in application-specific pointcuts.

#### Static pointcuts

Static pointcuts are based on method and target class, and cannot take into account the method’s arguments. Static pointcuts are sufficient - *and best* - for most usages. It’s possible for Spring to evaluate a static pointcut only once, when a method is first invoked: after that, there is no need to evaluate the pointcut again with each method invocation.

Let’s consider some static pointcut implementations included with Spring.

##### Regular expression pointcuts

One obvious way to specify static pointcuts is regular expressions. Several AOP frameworks besides Spring make this possible.`org.springframework.aop.support.JdkRegexpMethodPointcut` is a generic regular expression pointcut, using the regular expression support in JDK 1.4+.

Using the `JdkRegexpMethodPointcut` class, you can provide a list of pattern Strings. If any of these is a match, the pointcut will evaluate to true. (So the result is effectively the union of these pointcuts.)

The usage is shown below:

```xml
<bean id="settersAndAbsquatulatePointcut"
		class="org.springframework.aop.support.JdkRegexpMethodPointcut">
	<property name="patterns">
		<list>
			<value>.*set.*</value>
			<value>.*absquatulate</value>
		</list>
	</property>
</bean>
```

Spring provides a convenience class, `RegexpMethodPointcutAdvisor`, that allows us to also reference an Advice (remember that an Advice can be an interceptor, before advice, throws advice etc.). Behind the scenes, Spring will use a `JdkRegexpMethodPointcut`. Using `RegexpMethodPointcutAdvisor` simplifies wiring, as the one bean encapsulates both pointcut and advice, as shown below:

```xml
<bean id="settersAndAbsquatulateAdvisor"
		class="org.springframework.aop.support.RegexpMethodPointcutAdvisor">
	<property name="advice">
		<ref bean="beanNameOfAopAllianceInterceptor"/>
	</property>
	<property name="patterns">
		<list>
			<value>.*set.*</value>
			<value>.*absquatulate</value>
		</list>
	</property>
</bean>
```

*RegexpMethodPointcutAdvisor* can be used with any Advice type.

##### Attribute-driven pointcuts

An important type of static pointcut is a *metadata-driven* pointcut. This uses the values of metadata attributes: typically, source-level metadata.

#### Dynamic pointcuts

Dynamic pointcuts are costlier to evaluate than static pointcuts. They take into account method *arguments*, as well as static information. This means that they must be evaluated with every method invocation; the result cannot be cached, as arguments will vary.

The main example is the `control flow` pointcut.

##### Control flow pointcuts

Spring control flow pointcuts are conceptually similar to AspectJ *cflow* pointcuts, although less powerful. (There is currently no way to specify that a pointcut executes below a join point matched by another pointcut.) A control flow pointcut matches the current call stack. For example, it might fire if the join point was invoked by a method in the `com.mycompany.web` package, or by the `SomeCaller` class. Control flow pointcuts are specified using the `org.springframework.aop.support.ControlFlowPointcut` class.

| ![[Note]](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| ---------------------------------------- |
| Control flow pointcuts are significantly more expensive to evaluate at runtime than even other dynamic pointcuts. In Java 1.4, the cost is about 5 times that of other dynamic pointcuts. |

### 8.2.5 Pointcut superclasses

Spring provides useful pointcut superclasses to help you to implement your own pointcuts.

Because static pointcuts are most useful, you’ll probably subclass StaticMethodMatcherPointcut, as shown below. This requires implementing just one abstract method (although it’s possible to override other methods to customize behavior):

```java
class TestStaticPointcut extends StaticMethodMatcherPointcut {

	public boolean matches(Method m, Class targetClass) {
		// return true if custom criteria match
	}
}
```

There are also superclasses for dynamic pointcuts.

You can use custom pointcuts with any advice type in Spring 1.0 RC2 and above.

### 8.2.6 Custom pointcuts

Because pointcuts in Spring AOP are Java classes, rather than language features (as in AspectJ) it’s possible to declare custom pointcuts, whether static or dynamic. Custom pointcuts in Spring can be arbitrarily complex. However, using the AspectJ pointcut expression language is recommended if possible.

| ![[Note]](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| ---------------------------------------- |
| Later versions of Spring may offer support for "semantic pointcuts" as offered by JAC: for example, "all methods that change instance variables in the target object." |