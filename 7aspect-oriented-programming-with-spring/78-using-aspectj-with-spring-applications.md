## 7.8 Using AspectJ with Spring applications

Everything we’ve covered so far in this chapter is pure Spring AOP. In this section, we’re going to look at how you can use the AspectJ compiler/weaver instead of, or in addition to, Spring AOP if your needs go beyond the facilities offered by Spring AOP alone.

Spring ships with a small AspectJ aspect library, which is available standalone in your distribution as `spring-aspects.jar`; you’ll need to add this to your classpath in order to use the aspects in it. [Section 7.8.1, “Using AspectJ to dependency inject domain objects with Spring”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#aop-atconfigurable) and [Section 7.8.2, “Other Spring aspects for AspectJ”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#aop-ajlib-other)discuss the  content of this library and how you can use it. [Section 7.8.3, “Configuring AspectJ aspects using Spring IoC”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#aop-aj-configure) discusses how to dependency inject AspectJ aspects that are woven using the AspectJ compiler. Finally, [Section 7.8.4, “Load-time weaving with AspectJ in the Spring Framework”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#aop-aj-ltw) provides an introduction to load-time weaving for Spring applications using AspectJ.

### 7.8.1 Using AspectJ to dependency inject domain objects with Spring

The Spring container instantiates and configures beans defined in your application context. It is also possible to ask a bean factory to configure a *pre-existing* object given the name of a bean definition containing the configuration to be applied. The `spring-aspects.jar` contains an annotation-driven aspect that exploits this capability to allow dependency injection of *any object*. The support is intended to be used for objects created *outside of the control of any container*. Domain objects often fall into this category because they are often created programmatically using the `new` operator, or by an ORM tool as a result of a database query.

The `@Configurable` annotation marks a class as eligible for Spring-driven configuration. In the simplest case it can be used just as a marker annotation:

```java
package com.xyz.myapp.domain;

import org.springframework.beans.factory.annotation.Configurable;

@Configurable
public class Account {
	// ...
}
```

When used as a marker interface in this way, Spring will configure new instances of the annotated type ( `Account` in this case) using a bean definition (typically prototype-scoped) with the same name as the fully-qualified type name ( `com.xyz.myapp.domain.Account`). Since the default name for a bean is the fully-qualified name of its type, a convenient way to declare the prototype definition is simply to omit the `id` attribute:

```xml
<bean class="com.xyz.myapp.domain.Account" scope="prototype">
	<property name="fundsTransferService" ref="fundsTransferService"/>
</bean>
```

If you want to explicitly specify the name of the prototype bean definition to use, you can do so directly in the annotation:

```java
package com.xyz.myapp.domain;

import org.springframework.beans.factory.annotation.Configurable;

@Configurable("account")
public class Account {
	// ...
}
```

Spring will now look for a bean definition named "account" and use that as the definition to configure new `Account` instances.

You can also use autowiring to avoid having to specify a dedicated bean definition at all. To have Spring apply autowiring use the `autowire` property of the`@Configurable` annotation: specify either `@Configurable(autowire=Autowire.BY_TYPE)` or `@Configurable(autowire=Autowire.BY_NAME` for autowiring by type or by name respectively. As an alternative, as of Spring 2.5 it is preferable to specify explicit, annotation-driven dependency injection for your `@Configurable` beans by using `@Autowired` or `@Inject` at the field or method level (see [Section 3.9, “Annotation-based container configuration”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#beans-annotation-config) for further details).

Finally you can enable Spring dependency checking for the object references in the newly created and configured object by using the `dependencyCheck` attribute (for example: `@Configurable(autowire=Autowire.BY_NAME,dependencyCheck=true)`). If this attribute is set to true, then Spring will validate after configuration that all properties (*which are not primitives or collections*) have been set.

Using the annotation on its own does nothing of course. It is the `AnnotationBeanConfigurerAspect` in `spring-aspects.jar` that acts on the presence of the annotation. In essence the aspect says "after returning from the initialization of a new object of a type annotated with `@Configurable`, configure the newly created object using Spring in accordance with the properties of the annotation". In this context, *initialization* refers to newly instantiated objects (e.g., objects instantiated with the `new` operator) as well as to `Serializable` objects that are undergoing deserialization (e.g., via [readResolve()](https://docs.oracle.com/javase/6/docs/api/java/io/Serializable.html)).

| ![[Note]](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| ---------------------------------------- |
| One of the key phrases in the above paragraph is '*in essence*'. For most cases, the exact semantics of '*after returning from the initialization of a new object*' will be fine… in this context, '*after initialization*' means that the dependencies will be injected *after* the object has been constructed - this means that the dependencies will not be available for use in the constructor bodies of the class. If you want the dependencies to be injected *before* the constructor bodies execute, and thus be available for use in the body of the constructors, then you need to define this on the `@Configurable` declaration like so:`@Configurable(preConstruction=true)`You can find out more information about the language semantics of the various pointcut types in AspectJ [in this appendix](https://www.eclipse.org/aspectj/doc/next/progguide/semantics-joinPoints.html) of the [AspectJ Programming Guide](https://www.eclipse.org/aspectj/doc/next/progguide/index.html). |

For this to work the annotated types must be woven with the AspectJ weaver - you can either use a build-time Ant or Maven task to do this (see for example the [AspectJ Development Environment Guide](https://www.eclipse.org/aspectj/doc/released/devguide/antTasks.html)) or load-time weaving (see [Section 7.8.4, “Load-time weaving with AspectJ in the Spring Framework”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#aop-aj-ltw)). The`AnnotationBeanConfigurerAspect` itself needs configuring by Spring (in order to obtain a reference to the bean factory that is to be used to configure new objects). If you are using Java based configuration simply add `@EnableSpringConfigured` to any `@Configuration` class.

```java
@Configuration
@EnableSpringConfigured
public class AppConfig {

}
```

If you prefer XML based configuration, the Spring [`context` namespace](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#xsd-config-body-schemas-context) defines a convenient `context:spring-configured` element:

```xml
<context:spring-configured/>
```

Instances of `@Configurable` objects created *before* the aspect has been configured will result in a message being issued to the debug log and no configuration of the object taking place. An example might be a bean in the Spring configuration that creates domain objects when it is initialized by Spring. In this case you can use the "depends-on" bean attribute to manually specify that the bean depends on the configuration aspect.

```xml
<bean id="myService"
		class="com.xzy.myapp.service.MyService"
		depends-on="org.springframework.beans.factory.aspectj.AnnotationBeanConfigurerAspect">

	<!-- ... -->

</bean>
```

| ![[Note]](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| ---------------------------------------- |
| Do not activate `@Configurable` processing through the bean configurer aspect unless you really mean to rely on its semantics at runtime. In particular, make sure that you do not use `@Configurable` on bean classes which are registered as regular Spring beans with the container: You would get double initialization otherwise, once through the container and once through the aspect. |

#### Unit testing @Configurable objects

One of the goals of the `@Configurable` support is to enable independent unit testing of domain objects without the difficulties associated with hard-coded lookups. If`@Configurable` types have not been woven by AspectJ then the annotation has no affect during unit testing, and you can simply set mock or stub property references in the object under test and proceed as normal. If `@Configurable` types *have* been woven by AspectJ then you can still unit test outside of the container as normal, but you will see a warning message each time that you construct an `@Configurable` object indicating that it has not been configured by Spring.

#### Working with multiple application contexts

The `AnnotationBeanConfigurerAspect` used to implement the `@Configurable` support is an AspectJ singleton aspect. The scope of a singleton aspect is the same as the scope of `static` members, that is to say there is one aspect instance per classloader that defines the type. This means that if you define multiple application contexts within the same classloader hierarchy you need to consider where to define the `@EnableSpringConfigured` bean and where to place `spring-aspects.jar`on the classpath.

Consider a typical Spring web-app configuration with a shared parent application context defining common business services and everything needed to support them, and one child application context per servlet containing definitions particular to that servlet. All of these contexts will co-exist within the same classloader hierarchy, and so the `AnnotationBeanConfigurerAspect` can only hold a reference to one of them. In this case we recommend defining the `@EnableSpringConfigured` bean in the shared (parent) application context: this defines the services that you are likely to want to inject into domain objects. A consequence is that you cannot configure domain objects with references to beans defined in the child (servlet-specific) contexts using the @Configurable mechanism (probably not something you want to do anyway!).

When deploying multiple web-apps within the same container, ensure that each web-application loads the types in `spring-aspects.jar` using its own classloader (for example, by placing `spring-aspects.jar` in `'WEB-INF/lib'`). If `spring-aspects.jar` is only added to the container wide classpath (and hence loaded by the shared parent classloader), all web applications will share the same aspect instance which is probably not what you want.

### 7.8.2 Other Spring aspects for AspectJ

In addition to the `@Configurable` aspect, `spring-aspects.jar` contains an AspectJ aspect that can be used to drive Spring’s transaction management for types and methods annotated with the `@Transactional` annotation. This is primarily intended for users who want to use the Spring Framework’s transaction support outside of the Spring container.

The aspect that interprets `@Transactional` annotations is the `AnnotationTransactionAspect`. When using this aspect, you must annotate the *implementation* class (and/or methods within that class), *not* the interface (if any) that the class implements. AspectJ follows Java’s rule that annotations on interfaces are *not inherited*.

A `@Transactional` annotation on a class specifies the default transaction semantics for the execution of any *public* operation in the class.

A `@Transactional` annotation on a method within the class overrides the default transaction semantics given by the class annotation (if present). Methods of any visibility may be annotated, including private methods. Annotating non-public methods directly is the only way to get transaction demarcation for the execution of such methods.

| ![[Tip]](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/images/tip.png.pagespeed.ce.w22Wv-tZ37.png) |
| ---------------------------------------- |
| Since Spring Framework 4.2, `spring-aspects` provides a similar aspect that offers the exact same features for the standard `javax.transaction.Transactional` annotation. Check `JtaAnnotationTransactionAspect` for more details. |

For AspectJ programmers that want to use the Spring configuration and transaction management support but don’t want to (or cannot) use annotations, `spring-aspects.jar` also contains `abstract` aspects you can extend to provide your own pointcut definitions. See the sources for the `AbstractBeanConfigurerAspect` and `AbstractTransactionAspect` aspects for more information. As an example, the following excerpt shows how you could write an aspect to configure all instances of objects defined in the domain model using prototype bean definitions that match the fully-qualified class names:

```java
public aspect DomainObjectConfiguration extends AbstractBeanConfigurerAspect {

	public DomainObjectConfiguration() {
		setBeanWiringInfoResolver(new ClassNameBeanWiringInfoResolver());
	}

	// the creation of a new bean (any object in the domain model)
	protected pointcut beanCreation(Object beanInstance) :
		initialization(new(..)) &&
		SystemArchitecture.inDomainModel() &&
		this(beanInstance);

}
```

### 7.8.3 Configuring AspectJ aspects using Spring IoC

When using AspectJ aspects with Spring applications, it is natural to both want and expect to be able to configure such aspects using Spring. The AspectJ runtime itself is responsible for aspect creation, and the means of configuring the AspectJ created aspects via Spring depends on the AspectJ instantiation model (the `per-xxx`clause) used by the aspect.

The majority of AspectJ aspects are *singleton* aspects. Configuration of these aspects is very easy: simply create a bean definition referencing the aspect type as normal, and include the bean attribute `'factory-method="aspectOf"'`. This ensures that Spring obtains the aspect instance by asking AspectJ for it rather than trying to create an instance itself. For example:

```xml
<bean id="profiler" class="com.xyz.profiler.Profiler"
		factory-method="aspectOf">

	<property name="profilingStrategy" ref="jamonProfilingStrategy"/>
</bean>
```

Non-singleton aspects are harder to configure: however it is possible to do so by creating prototype bean definitions and using the `@Configurable` support from`spring-aspects.jar` to configure the aspect instances once they have bean created by the AspectJ runtime.

If you have some @AspectJ aspects that you want to weave with AspectJ (for example, using load-time weaving for domain model types) and other @AspectJ aspects that you want to use with Spring AOP, and these aspects are all configured using Spring, then you will need to tell the Spring AOP @AspectJ autoproxying support which exact subset of the @AspectJ aspects defined in the configuration should be used for autoproxying. You can do this by using one or more `<include/>` elements inside the `<aop:aspectj-autoproxy/>` declaration. Each `<include/>` element specifies a name pattern, and only beans with names matched by at least one of the patterns will be used for Spring AOP autoproxy configuration:

```xml
<aop:aspectj-autoproxy>
	<aop:include name="thisBean"/>
	<aop:include name="thatBean"/>
</aop:aspectj-autoproxy>
```

| ![[Note]](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| ---------------------------------------- |
| Do not be misled by the name of the `<aop:aspectj-autoproxy/>` element: using it will result in the creation of *Spring AOP proxies*. The @AspectJ style of aspect declaration is just being used here, but the AspectJ runtime is *not* involved. |

### 7.8.4 Load-time weaving with AspectJ in the Spring Framework

Load-time weaving (LTW) refers to the process of weaving AspectJ aspects into an application’s class files as they are being loaded into the Java virtual machine (JVM). The focus of this section is on configuring and using LTW in the specific context of the Spring Framework: this section is not an introduction to LTW though. For full details on the specifics of LTW and configuring LTW with just AspectJ (with Spring not being involved at all), see the [LTW section of the AspectJ Development Environment Guide](https://www.eclipse.org/aspectj/doc/released/devguide/ltw.html).

The value-add that the Spring Framework brings to AspectJ LTW is in enabling much finer-grained control over the weaving process. 'Vanilla' AspectJ LTW is effected using a Java (5+) agent, which is switched on by specifying a VM argument when starting up a JVM. It is thus a JVM-wide setting, which may be fine in some situations, but often is a little too coarse. Spring-enabled LTW enables you to switch on LTW on a *per-ClassLoader* basis, which obviously is more fine-grained and which can make more sense in a 'single-JVM-multiple-application' environment (such as is found in a typical application server environment).

Further, [in certain environments](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#aop-aj-ltw-environments), this support enables load-time weaving *without making any modifications to the application server’s launch script* that will be needed to add `-javaagent:path/to/aspectjweaver.jar` or (as we describe later in this section) `-javaagent:path/to/org.springframework.instrument-{version}.jar`(previously named `spring-agent.jar`). Developers simply modify one or more files that form the application context to enable load-time weaving instead of relying on administrators who typically are in charge of the deployment configuration such as the launch script.

Now that the sales pitch is over, let us first walk through a quick example of AspectJ LTW using Spring, followed by detailed specifics about elements introduced in the following example. For a complete example, please see the [Petclinic sample application](https://github.com/spring-projects/spring-petclinic).

#### A first example

Let us assume that you are an application developer who has been tasked with diagnosing the cause of some performance problems in a system. Rather than break out a profiling tool, what we are going to do is switch on a simple profiling aspect that will enable us to very quickly get some performance metrics, so that we can then apply a finer-grained profiling tool to that specific area immediately afterwards.

| ![[Note]](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| ---------------------------------------- |
| The example presented here uses XML style configuration, it is also possible to configure and use @AspectJ with [Java Configuration](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#beans-java). Specifically the`@EnableLoadTimeWeaving` annotation can be used as an alternative to `<context:load-time-weaver/>` (see [below](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#aop-aj-ltw-spring) for details). |

Here is the profiling aspect. Nothing too fancy, just a quick-and-dirty time-based profiler, using the @AspectJ-style of aspect declaration.

```java
package foo;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Pointcut;
import org.springframework.util.StopWatch;
import org.springframework.core.annotation.Order;

@Aspect
public class ProfilingAspect {

	@Around("methodsToBeProfiled()")
	public Object profile(ProceedingJoinPoint pjp) throws Throwable {
		StopWatch sw = new StopWatch(getClass().getSimpleName());
		try {
			sw.start(pjp.getSignature().getName());
			return pjp.proceed();
		} finally {
			sw.stop();
			System.out.println(sw.prettyPrint());
		}
	}

	@Pointcut("execution(public * foo..*.*(..))")
	public void methodsToBeProfiled(){}
}
```

We will also need to create an `META-INF/aop.xml` file, to inform the AspectJ weaver that we want to weave our `ProfilingAspect` into our classes. This file convention, namely the presence of a file (or files) on the Java classpath called `META-INF/aop.xml` is standard AspectJ.

```xml
<!DOCTYPE aspectj PUBLIC "-//AspectJ//DTD//EN" "http://www.eclipse.org/aspectj/dtd/aspectj.dtd">
<aspectj>

	<weaver>
		<!-- only weave classes in our application-specific packages -->
		<include within="foo.*"/>
	</weaver>

	<aspects>
		<!-- weave in just this aspect -->
		<aspect name="foo.ProfilingAspect"/>
	</aspects>

</aspectj>
```

Now to the Spring-specific portion of the configuration. We need to configure a `LoadTimeWeaver` (all explained later, just take it on trust for now). This load-time weaver is the essential component responsible for weaving the aspect configuration in one or more `META-INF/aop.xml` files into the classes in your application. The good thing is that it does not require a lot of configuration, as can be seen below (there are some more options that you can specify, but these are detailed later).

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="
		http://www.springframework.org/schema/beans
		http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context
		http://www.springframework.org/schema/context/spring-context.xsd">

	<!-- a service object; we will be profiling its methods -->
	<bean id="entitlementCalculationService"
			class="foo.StubEntitlementCalculationService"/>

	<!-- this switches on the load-time weaving -->
	<context:load-time-weaver/>
</beans>
```

Now that all the required artifacts are in place - the aspect, the `META-INF/aop.xml` file, and the Spring configuration -, let us create a simple driver class with a`main(..)` method to demonstrate the LTW in action.

```java
package foo;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public final class Main {

	public static void main(String[] args) {

		ApplicationContext ctx = new ClassPathXmlApplicationContext("beans.xml", Main.class);

		EntitlementCalculationService entitlementCalculationService
			= (EntitlementCalculationService) ctx.getBean("entitlementCalculationService");

		// the profiling aspect is 'woven' around this method execution
		entitlementCalculationService.calculateEntitlement();
	}
}
```

There is one last thing to do. The introduction to this section did say that one could switch on LTW selectively on a per- `ClassLoader` basis with Spring, and this is true. However, just for this example, we are going to use a Java agent (supplied with Spring) to switch on the LTW. This is the command line we will use to run the above `Main` class:

```bash
java -javaagent:C:/projects/foo/lib/global/spring-instrument.jar foo.Main
```

The `-javaagent` is a flag for specifying and enabling [agents to instrument programs running on the JVM](https://docs.oracle.com/javase/6/docs/api/java/lang/instrument/package-summary.html). The Spring Framework ships with such an agent, the `InstrumentationSavingAgent`, which is packaged in the `spring-instrument.jar` that was supplied as the value of the `-javaagent` argument in the above example.

The output from the execution of the `Main` program will look something like that below. (I have introduced a `Thread.sleep(..)` statement into the `calculateEntitlement()` implementation so that the profiler actually captures something other than 0 milliseconds - the `01234` milliseconds is *not* an overhead introduced by the AOP :) )

```java
Calculating entitlement

StopWatch 'ProfilingAspect': running time (millis) = 1234
------ ----- ----------------------------
ms     %     Task name
------ ----- ----------------------------
01234  100%  calculateEntitlement
```

Since this LTW is effected using full-blown AspectJ, we are not just limited to advising Spring beans; the following slight variation on the `Main` program will yield the same result.

```java
package foo;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public final class Main {

	public static void main(String[] args) {

		new ClassPathXmlApplicationContext("beans.xml", Main.class);

		EntitlementCalculationService entitlementCalculationService =
			new StubEntitlementCalculationService();

		// the profiling aspect will be 'woven' around this method execution
		entitlementCalculationService.calculateEntitlement();
	}
}
```

Notice how in the above program we are simply bootstrapping the Spring container, and then creating a new instance of the `StubEntitlementCalculationService`totally outside the context of Spring… the profiling advice still gets woven in.

The example admittedly is simplistic… however the basics of the LTW support in Spring have all been introduced in the above example, and the rest of this section will explain the 'why' behind each bit of configuration and usage in detail.

| ![[Note]](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| ---------------------------------------- |
| The `ProfilingAspect` used in this example may be basic, but it is quite useful. It is a nice example of a development-time aspect that developers can use during development (of course), and then quite easily exclude from builds of the application being deployed into UAT or production. |

#### Aspects

The aspects that you use in LTW have to be AspectJ aspects. They can be written in either the AspectJ language itself or you can write your aspects in the @AspectJ-style. It means that your aspects are then both valid AspectJ *and* Spring AOP aspects. Furthermore, the compiled aspect classes need to be available on the classpath.

#### 'META-INF/aop.xml'

The AspectJ LTW infrastructure is configured using one or more `META-INF/aop.xml` files, that are on the Java classpath (either directly, or more typically in jar files).

The structure and contents of this file is detailed in the main AspectJ reference documentation, and the interested reader is [referred to that resource](https://www.eclipse.org/aspectj/doc/released/devguide/ltw-configuration.html). (I appreciate that this section is brief, but the `aop.xml` file is 100% AspectJ - there is no Spring-specific information or semantics that apply to it, and so there is no extra value that I can contribute either as a result), so rather than rehash the quite satisfactory section that the AspectJ developers wrote, I am just directing you there.)

#### Required libraries (JARS)

At a minimum you will need the following libraries to use the Spring Framework’s support for AspectJ LTW:

- `spring-aop.jar` (version 2.5 or later, plus all mandatory dependencies)
- `aspectjweaver.jar` (version 1.6.8 or later)

If you are using the [Spring-provided agent to enable instrumentation](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#aop-aj-ltw-environment-generic), you will also need:

- `spring-instrument.jar`

#### Spring configuration

The key component in Spring’s LTW support is the `LoadTimeWeaver` interface (in the `org.springframework.instrument.classloading` package), and the numerous implementations of it that ship with the Spring distribution. A `LoadTimeWeaver` is responsible for adding one or more `java.lang.instrument.ClassFileTransformers` to a `ClassLoader` at runtime, which opens the door to all manner of interesting applications, one of which happens to be the LTW of aspects.

| ![[Tip]](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/images/tip.png.pagespeed.ce.w22Wv-tZ37.png) |
| ---------------------------------------- |
| If you are unfamiliar with the idea of runtime class file transformation, you are encouraged to read the javadoc API documentation for the `java.lang.instrument` package before continuing. This is not a huge chore because there is - rather annoyingly - precious little documentation there… the key interfaces and classes will at least be laid out in front of you for reference as you read through this section. |

Configuring a `LoadTimeWeaver` for a particular `ApplicationContext` can be as easy as adding one line. (Please note that you almost certainly will need to be using an `ApplicationContext` as your Spring container - typically a `BeanFactory` will not be enough because the LTW support makes use of `BeanFactoryPostProcessors`.)

To enable the Spring Framework’s LTW support, you need to configure a `LoadTimeWeaver`, which typically is done using the `@EnableLoadTimeWeaving` annotation.

```java
@Configuration
@EnableLoadTimeWeaving
public class AppConfig {

}
```

Alternatively, if you prefer XML based configuration, use the `<context:load-time-weaver/>` element. Note that the element is defined in the `context` namespace.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="
		http://www.springframework.org/schema/beans
		http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context
		http://www.springframework.org/schema/context/spring-context.xsd">

	<context:load-time-weaver/>

</beans>
```

The above configuration will define and register a number of LTW-specific infrastructure beans for you automatically, such as a `LoadTimeWeaver` and an `AspectJWeavingEnabler`. The default `LoadTimeWeaver` is the `DefaultContextLoadTimeWeaver` class, which attempts to decorate an automatically detected `LoadTimeWeaver`: the exact type of `LoadTimeWeaver` that will be 'automatically detected' is dependent upon your runtime environment (summarized in the following table).

**Table 7.1. DefaultContextLoadTimeWeaver LoadTimeWeavers**

| Runtime Environment                      | `LoadTimeWeaver`implementation  |
| ---------------------------------------- | ------------------------------- |
| Running in Oracle’s [WebLogic](http://www.oracle.com/technetwork/middleware/weblogic/overview/index-085209.html) | `WebLogicLoadTimeWeaver`        |
| Running in Oracle’s [GlassFish](http://glassfish.dev.java.net/) | `GlassFishLoadTimeWeaver`       |
| Running in [Apache Tomcat](https://tomcat.apache.org/) | `TomcatLoadTimeWeaver`          |
| Running in Red Hat’s [JBoss AS](https://www.jboss.org/jbossas/) or [WildFly](http://www.wildfly.org/) | `JBossLoadTimeWeaver`           |
| Running in IBM’s [WebSphere](https://www-01.ibm.com/software/webservers/appserv/was/) | `WebSphereLoadTimeWeaver`       |
| JVM started with Spring `InstrumentationSavingAgent` *(java -javaagent:path/to/spring-instrument.jar)* | `InstrumentationLoadTimeWeaver` |
| Fallback, expecting the underlying ClassLoader to follow common conventions (e.g. applicable to `TomcatInstrumentableClassLoader` and [Resin](http://www.caucho.com/)) | `ReflectiveLoadTimeWeaver`      |

Note that these are just the `LoadTimeWeavers` that are autodetected when using the `DefaultContextLoadTimeWeaver`: it is of course possible to specify exactly which `LoadTimeWeaver` implementation that you wish to use.

To specify a specific `LoadTimeWeaver` with Java configuration implement the `LoadTimeWeavingConfigurer` interface and override the `getLoadTimeWeaver()`method:

```java
@Configuration
@EnableLoadTimeWeaving
public class AppConfig implements LoadTimeWeavingConfigurer {

	@Override
	public LoadTimeWeaver getLoadTimeWeaver() {
		return new ReflectiveLoadTimeWeaver();
	}
}
```

If you are using XML based configuration you can specify the fully-qualified classname as the value of the `weaver-class` attribute on the `<context:load-time-weaver/>` element:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="
		http://www.springframework.org/schema/beans
		http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context
		http://www.springframework.org/schema/context/spring-context.xsd">

	<context:load-time-weaver
			weaver-class="org.springframework.instrument.classloading.ReflectiveLoadTimeWeaver"/>

</beans>
```

The `LoadTimeWeaver` that is defined and registered by the configuration can be later retrieved from the Spring container using the well-known name `loadTimeWeaver`. Remember that the `LoadTimeWeaver` exists just as a mechanism for Spring’s LTW infrastructure to add one or more `ClassFileTransformers`. The actual`ClassFileTransformer` that does the LTW is the `ClassPreProcessorAgentAdapter` (from the `org.aspectj.weaver.loadtime` package) class. See the class-level javadocs of the `ClassPreProcessorAgentAdapter` class for further details, because the specifics of how the weaving is actually effected is beyond the scope of this section.

There is one final attribute of the configuration left to discuss: the `aspectjWeaving` attribute (or `aspectj-weaving` if you are using XML). This is a simple attribute that controls whether LTW is enabled or not; it is as simple as that. It accepts one of three possible values, summarized below, with the default value being `autodetect` if the attribute is not present.

**Table 7.2. AspectJ weaving attribute values**

| Annotation Value | XML Value    | Explanation                              |
| ---------------- | ------------ | ---------------------------------------- |
| `ENABLED`        | `on`         | AspectJ weaving is on, and aspects will be woven at load-time as appropriate. |
| `DISABLED`       | `off`        | LTW is off… no aspect will be woven at load-time. |
| `AUTODETECT`     | `autodetect` | If the Spring LTW infrastructure can find at least one `META-INF/aop.xml` file, then AspectJ weaving is on, else it is off. This is the default value. |

#### Environment-specific configuration

This last section contains any additional settings and configuration that you will need when using Spring’s LTW support in environments such as application servers and web containers.

##### Tomcat

Historically, [Apache Tomcat](https://tomcat.apache.org/)'s default class loader did not support class transformation which is why Spring provides an enhanced implementation that addresses this need. Named `TomcatInstrumentableClassLoader`, the loader works on Tomcat 6.0 and above.

| ![[Tip]](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/images/tip.png.pagespeed.ce.w22Wv-tZ37.png) |
| ---------------------------------------- |
| Do not define `TomcatInstrumentableClassLoader` anymore on Tomcat 8.0 and higher. Instead, let Spring automatically use Tomcat’s new native `InstrumentableClassLoader` facility through the `TomcatLoadTimeWeaver` strategy. |

If you still need to use `TomcatInstrumentableClassLoader`, it can be registered individually for *each* web application as follows:

- Copy `org.springframework.instrument.tomcat.jar` into *$CATALINA_HOME*/lib, where *$CATALINA_HOME* represents the root of the Tomcat installation)
- Instruct Tomcat to use the custom class loader (instead of the default) by editing the web application context file:

```xml
<Context path="/myWebApp" docBase="/my/webApp/location">
	<Loader
		loaderClass="org.springframework.instrument.classloading.tomcat.TomcatInstrumentableClassLoader"/>
</Context>
```

Apache Tomcat (6.0+) supports several context locations:

- server configuration file - *$CATALINA_HOME/conf/server.xml*
- default context configuration - *$CATALINA_HOME/conf/context.xml* - that affects all deployed web applications
- per-web application configuration which can be deployed either on the server-side at *$CATALINA_HOME/conf/[enginename]/[hostname]/[webapp]-context.xml* or embedded inside the web-app archive at *META-INF/context.xml*

For efficiency, the embedded per-web-app configuration style is recommended because it will impact only applications that use the custom class loader and does not require any changes to the server configuration. See the Tomcat 6.0.x [documentation](https://tomcat.apache.org/tomcat-6.0-doc/config/context.html) for more details about available context locations.

Alternatively, consider the use of the Spring-provided generic VM agent, to be specified in Tomcat’s launch script (see above). This will make instrumentation available to all deployed web applications, no matter what ClassLoader they happen to run on.

##### WebLogic, WebSphere, Resin, GlassFish, JBoss

Recent versions of WebLogic Server (version 10 and above), IBM WebSphere Application Server (version 7 and above), Resin (3.1 and above) and JBoss (6.x or above) provide a ClassLoader that is capable of local instrumentation. Spring’s native LTW leverages such ClassLoaders to enable AspectJ weaving. You can enable LTW by simply activating load-time weaving as described earlier. Specifically, you do *not* need to modify the launch script to add `-javaagent:path/to/spring-instrument.jar`.

Note that GlassFish instrumentation-capable ClassLoader is available only in its EAR environment. For GlassFish web applications, follow the Tomcat setup instructions as outlined above.

Note that on JBoss 6.x, the app server scanning needs to be disabled to prevent it from loading the classes before the application actually starts. A quick workaround is to add to your artifact a file named `WEB-INF/jboss-scanning.xml` with the following content:

```xml
<scanning xmlns="urn:jboss:scanning:1.0"/>
```

##### Generic Java applications

When class instrumentation is required in environments that do not support or are not supported by the existing `LoadTimeWeaver` implementations, a JDK agent can be the only solution. For such cases, Spring provides `InstrumentationLoadTimeWeaver`, which requires a Spring-specific (but very general) VM agent,`org.springframework.instrument-{version}.jar` (previously named `spring-agent.jar`).

To use it, you must start the virtual machine with the Spring agent, by supplying the following JVM options:

```bash
-javaagent:/path/to/org.springframework.instrument-{version}.jar
```

Note that this requires modification of the VM launch script which may prevent you from using this in application server environments (depending on your operation policies). Additionally, the JDK agent will instrument the *entire* VM which can prove expensive.

For performance reasons, it is recommended to use this configuration only if your target environment (such as [Jetty](https://www.eclipse.org/jetty/)) does not have (or does not support) a dedicated LTW.