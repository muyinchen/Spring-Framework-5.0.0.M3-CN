## 11.5 Spring TestContext Framework

The *Spring TestContext Framework* (located in the `org.springframework.test.context` package) provides generic, annotation-driven unit and integration testing support that is agnostic of the testing framework in use. The TestContext framework also places a great deal of importance on *convention over configuration* with reasonable defaults that can be overridden through annotation-based configuration.

In addition to generic testing infrastructure, the TestContext framework provides explicit support for JUnit 4 and TestNG in the form of `abstract` support classes. For JUnit 4, Spring also provides a custom JUnit `Runner` and custom JUnit `Rules` that allow one to write so-called *POJO test classes*. POJO test classes are not required to extend a particular class hierarchy.

The following section provides an overview of the internals of the TestContext framework. If you are only interested in *using* the framework and not necessarily interested in *extending* it with your own custom listeners or custom loaders, feel free to go directly to the configuration ([context management](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#testcontext-ctx-management), [dependency injection](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#testcontext-fixture-di), [transaction management](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#testcontext-tx)), [support classes](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#testcontext-support-classes), and [annotation support](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#integration-testing-annotations) sections.

### 11.5.1 Key abstractions

The core of the framework consists of the `TestContextManager` class and the `TestContext`, `TestExecutionListener`, and `SmartContextLoader` interfaces. A`TestContextManager` is created per test class (e.g., for the execution of all test methods within a single test class in JUnit 4). The `TestContextManager` in turn manages a `TestContext` that holds the context of the current test. The `TestContextManager` also updates the state of the `TestContext` as the test progresses and delegates to `TestExecutionListener` implementations, which instrument the actual test execution by providing dependency injection, managing transactions, and so on. A `SmartContextLoader` is responsible for loading an `ApplicationContext` for a given test class. Consult the javadocs and the Spring test suite for further information and examples of various implementations.

#### TestContext

`TestContext` encapsulates the context in which a test is executed, agnostic of the actual testing framework in use, and provides context management and caching support for the test instance for which it is responsible. The `TestContext` also delegates to a `SmartContextLoader` to load an `ApplicationContext` if requested.

#### TestContextManager

`TestContextManager` is the main entry point into the *Spring TestContext Framework* and is responsible for managing a single `TestContext` and signaling events to each registered `TestExecutionListener` at well-defined test execution points:

- prior to any *before class* or *before all* methods of a particular testing framework
- test instance post-processing
- prior to any *before* or *before each* methods of a particular testing framework
- immediately before execution of the test method but after test setup
- immediately after execution of the test method but before test tear down
- after any *after* or *after each* methods of a particular testing framework
- after any *after class* or *after all* methods of a particular testing framework

#### TestExecutionListener

`TestExecutionListener` defines the API for reacting to test execution events published by the `TestContextManager` with which the listener is registered. See[Section 11.5.3, “TestExecutionListener configuration”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#testcontext-tel-config).

#### Context Loaders

`ContextLoader` is a strategy interface that was introduced in Spring 2.5 for loading an `ApplicationContext` for an integration test managed by the Spring TestContext Framework. Implement `SmartContextLoader` instead of this interface in order to provide support for annotated classes, active bean definition profiles, test property sources, context hierarchies, and `WebApplicationContext` support.

`SmartContextLoader` is an extension of the `ContextLoader` interface introduced in Spring 3.1. The `SmartContextLoader` SPI supersedes the `ContextLoader` SPI that was introduced in Spring 2.5. Specifically, a `SmartContextLoader` can choose to process resource `locations`, annotated `classes`, or context `initializers`. Furthermore, a `SmartContextLoader` can set active bean definition profiles and test property sources in the context that it loads.

Spring provides the following implementations:

- `DelegatingSmartContextLoader`: one of two default loaders which delegates internally to an `AnnotationConfigContextLoader`, a `GenericXmlContextLoader`, or a `GenericGroovyXmlContextLoader` depending either on the configuration declared for the test class or on the presence of default locations or default configuration classes. Groovy support is only enabled if Groovy is on the classpath.
- `WebDelegatingSmartContextLoader`: one of two default loaders which delegates internally to an `AnnotationConfigWebContextLoader`, a `GenericXmlWebContextLoader`, or a `GenericGroovyXmlWebContextLoader` depending either on the configuration declared for the test class or on the presence of default locations or default configuration classes. A web `ContextLoader` will only be used if `@WebAppConfiguration` is present on the test class. Groovy support is only enabled if Groovy is on the classpath.
- `AnnotationConfigContextLoader`: loads a standard `ApplicationContext` from *annotated classes*.
- `AnnotationConfigWebContextLoader`: loads a `WebApplicationContext` from *annotated classes*.
- `GenericGroovyXmlContextLoader`: loads a standard `ApplicationContext` from *resource locations* that are either Groovy scripts or XML configuration files.
- `GenericGroovyXmlWebContextLoader`: loads a `WebApplicationContext` from *resource locations* that are either Groovy scripts or XML configuration files.
- `GenericXmlContextLoader`: loads a standard `ApplicationContext` from XML *resource locations*.
- `GenericXmlWebContextLoader`: loads a `WebApplicationContext` from XML *resource locations*.
- `GenericPropertiesContextLoader`: loads a standard `ApplicationContext` from Java Properties files.

### 11.5.2 Bootstrapping the TestContext framework

The default configuration for the internals of the Spring TestContext Framework is sufficient for all common use cases. However, there are times when a development team or third party framework would like to change the default `ContextLoader`, implement a custom `TestContext` or `ContextCache`, augment the default sets of`ContextCustomizerFactory` and `TestExecutionListener` implementations, etc. For such low level control over how the TestContext framework operates, Spring provides a bootstrapping strategy.

`TestContextBootstrapper` defines the SPI for *bootstrapping* the TestContext framework. A `TestContextBootstrapper` is used by the `TestContextManager` to load the `TestExecutionListener` implementations for the current test and to build the `TestContext` that it manages. A custom bootstrapping strategy can be configured for a test class (or test class hierarchy) via `@BootstrapWith`, either directly or as a meta-annotation. If a bootstrapper is not explicitly configured via `@BootstrapWith`, either the `DefaultTestContextBootstrapper` or the `WebTestContextBootstrapper` will be used, depending on the presence of `@WebAppConfiguration`.

Since the `TestContextBootstrapper` SPI is likely to change in the future in order to accommodate new requirements, implementers are strongly encouraged not to implement this interface directly but rather to extend `AbstractTestContextBootstrapper` or one of its concrete subclasses instead.

### 11.5.3 TestExecutionListener configuration

Spring provides the following `TestExecutionListener` implementations that are registered by default, exactly in this order.

- `ServletTestExecutionListener`: configures Servlet API mocks for a `WebApplicationContext`
- `DirtiesContextBeforeModesTestExecutionListener`: handles the `@DirtiesContext` annotation for *before* modes
- `DependencyInjectionTestExecutionListener`: provides dependency injection for the test instance
- `DirtiesContextTestExecutionListener`: handles the `@DirtiesContext` annotation for *after* modes
- `TransactionalTestExecutionListener`: provides transactional test execution with default rollback semantics
- `SqlScriptsTestExecutionListener`: executes SQL scripts configured via the `@Sql` annotation

#### Registering custom TestExecutionListeners

Custom `TestExecutionListener`s can be registered for a test class and its subclasses via the `@TestExecutionListeners` annotation. See [annotation support](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#integration-testing-annotations) and the javadocs for `@TestExecutionListeners` for details and examples.

#### Automatic discovery of default TestExecutionListeners

Registering custom `TestExecutionListener`s via `@TestExecutionListeners` is suitable for custom listeners that are used in limited testing scenarios; however, it can become cumbersome if a custom listener needs to be used across a test suite. Since Spring Framework 4.1, this issue is addressed via support for automatic discovery of *default* `TestExecutionListener` implementations via the `SpringFactoriesLoader` mechanism.

Specifically, the `spring-test` module declares all core default `TestExecutionListener`s under the`org.springframework.test.context.TestExecutionListener` key in its `META-INF/spring.factories` properties file. Third-party frameworks and developers can contribute their own `TestExecutionListener`s to the list of default listeners in the same manner via their own `META-INF/spring.factories` properties file.

#### Ordering TestExecutionListeners

When the TestContext framework discovers default `TestExecutionListener`s via the aforementioned `SpringFactoriesLoader` mechanism, the instantiated listeners are sorted using Spring’s `AnnotationAwareOrderComparator` which honors Spring’s `Ordered` interface and `@Order` annotation for ordering. `AbstractTestExecutionListener` and all default `TestExecutionListener`s provided by Spring implement `Ordered` with appropriate values. Third-party frameworks and developers should therefore make sure that their *default* `TestExecutionListener`s are registered in the proper order by implementing `Ordered` or declaring `@Order`. Consult the javadocs for the `getOrder()` methods of the core default `TestExecutionListener`s for details on what values are assigned to each core listener.

#### Merging TestExecutionListeners

If a custom `TestExecutionListener` is registered via `@TestExecutionListeners`, the *default* listeners will not be registered. In most common testing scenarios, this effectively forces the developer to manually declare all default listeners in addition to any custom listeners. The following listing demonstrates this style of configuration.

```
@ContextConfiguration
@TestExecutionListeners({
	MyCustomTestExecutionListener.class,
	ServletTestExecutionListener.class,
	DirtiesContextBeforeModesTestExecutionListener.class,
	DependencyInjectionTestExecutionListener.class,
	DirtiesContextTestExecutionListener.class,
	TransactionalTestExecutionListener.class,
	SqlScriptsTestExecutionListener.class
})
public class MyTest {
	// class body...
}
```

The challenge with this approach is that it requires that the developer know exactly which listeners are registered by default. Moreover, the set of default listeners can change from release to release — for example, `SqlScriptsTestExecutionListener` was introduced in Spring Framework 4.1, and `DirtiesContextBeforeModesTestExecutionListener` was introduced in Spring Framework 4.2. Furthermore, third-party frameworks like Spring Security register their own default `TestExecutionListener`s via the aforementioned [automatic discovery mechanism](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#testcontext-tel-config-automatic-discovery).

To avoid having to be aware of and re-declare **all** *default* listeners, the `mergeMode` attribute of `@TestExecutionListeners` can be set to`MergeMode.MERGE_WITH_DEFAULTS`. `MERGE_WITH_DEFAULTS` indicates that locally declared listeners should be merged with the default listeners. The merging algorithm ensures that duplicates are removed from the list and that the resulting set of merged listeners is sorted according to the semantics of `AnnotationAwareOrderComparator` as described in [the section called “Ordering TestExecutionListeners”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#testcontext-tel-config-ordering). If a listener implements `Ordered` or is annotated with `@Order` it can influence the position in which it is merged with the defaults; otherwise, locally declared listeners will simply be appended to the list of default listeners when merged.

For example, if the `MyCustomTestExecutionListener` class in the previous example configures its `order` value (for example, `500`) to be less than the order of the`ServletTestExecutionListener` (which happens to be `1000`), the `MyCustomTestExecutionListener` can then be automatically merged with the list of defaults *in front of* the `ServletTestExecutionListener`, and the previous example could be replaced with the following.

```
@ContextConfiguration
@TestExecutionListeners(
	listeners = MyCustomTestExecutionListener.class,
	mergeMode = MERGE_WITH_DEFAULTS
)
public class MyTest {
	// class body...
}
```

### 11.5.4 Context management

Each `TestContext` provides context management and caching support for the test instance it is responsible for. Test instances do not automatically receive access to the configured `ApplicationContext`. However, if a test class implements the `ApplicationContextAware` interface, a reference to the `ApplicationContext` is supplied to the test instance. Note that `AbstractJUnit4SpringContextTests` and `AbstractTestNGSpringContextTests` implement `ApplicationContextAware`and therefore provide access to the `ApplicationContext` automatically.

| ![[Tip]](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/images/tip.png.pagespeed.ce.w22Wv-tZ37.png) |
| ---------------------------------------- |
| As an alternative to implementing the `ApplicationContextAware` interface, you can inject the application context for your test class through the `@Autowired` annotation on either a field or setter method. For example:`*@RunWith(SpringRunner.class)**@ContextConfiguration*public class MyTest {	**@Autowired**	private ApplicationContext applicationContext;	// class body...}`Similarly, if your test is configured to load a `WebApplicationContext`, you can inject the web application context into your test as follows:`*@RunWith(SpringRunner.class)***@WebAppConfiguration***@ContextConfiguration*public class MyWebAppTest {	**@Autowired**	private WebApplicationContext wac;	// class body...}`Dependency injection via `@Autowired` is provided by the `DependencyInjectionTestExecutionListener` which is configured by default (see[Section 11.5.5, “Dependency injection of test fixtures”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#testcontext-fixture-di)). |

Test classes that use the TestContext framework do not need to extend any particular class or implement a specific interface to configure their application context. Instead, configuration is achieved simply by declaring the `@ContextConfiguration` annotation at the class level. If your test class does not explicitly declare application context resource `locations` or annotated `classes`, the configured `ContextLoader` determines how to load a context from a default location or default configuration classes. In addition to context resource `locations` and annotated `classes`, an application context can also be configured via application context `initializers`.

The following sections explain how to configure an `ApplicationContext` via XML configuration files, Groovy scripts, annotated classes (typically `@Configuration`classes), or context initializers using Spring’s `@ContextConfiguration` annotation. Alternatively, you can implement and configure your own custom `SmartContextLoader` for advanced use cases.

#### Context configuration with XML resources

To load an `ApplicationContext` for your tests using XML configuration files, annotate your test class with `@ContextConfiguration` and configure the `locations`attribute with an array that contains the resource locations of XML configuration metadata. A plain or relative path — for example `"context.xml"` — will be treated as a classpath resource that is relative to the package in which the test class is defined. A path starting with a slash is treated as an absolute classpath location, for example`"/org/example/config.xml"`. A path which represents a resource URL (i.e., a path prefixed with `classpath:`, `file:`, `http:`, etc.) will be used *as is*.

```
@RunWith(SpringRunner.class)
// ApplicationContext will be loaded from "/app-config.xml" and
// "/test-config.xml" in the root of the classpath
@ContextConfiguration(locations={"/app-config.xml", "/test-config.xml"})
public class MyTest {
	// class body...
}
```

`@ContextConfiguration` supports an alias for the `locations` attribute through the standard Java `value` attribute. Thus, if you do not need to declare additional attributes in `@ContextConfiguration`, you can omit the declaration of the `locations` attribute name and declare the resource locations by using the shorthand format demonstrated in the following example.

```
@RunWith(SpringRunner.class)
@ContextConfiguration({"/app-config.xml", "/test-config.xml"})
public class MyTest {
	// class body...
}
```

If you omit both the `locations` and `value` attributes from the `@ContextConfiguration` annotation, the TestContext framework will attempt to detect a default XML resource location. Specifically, `GenericXmlContextLoader` and `GenericXmlWebContextLoader` detect a default location based on the name of the test class. If your class is named `com.example.MyTest`, `GenericXmlContextLoader` loads your application context from `"classpath:com/example/MyTest-context.xml"`.

```
package com.example;

@RunWith(SpringRunner.class)
// ApplicationContext will be loaded from
// "classpath:com/example/MyTest-context.xml"
@ContextConfiguration
public class MyTest {
	// class body...
}
```

#### Context configuration with Groovy scripts

To load an `ApplicationContext` for your tests using Groovy scripts that utilize the [Groovy Bean Definition DSL](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#groovy-bean-definition-dsl), annotate your test class with`@ContextConfiguration` and configure the `locations` or `value` attribute with an array that contains the resource locations of Groovy scripts. Resource lookup semantics for Groovy scripts are the same as those described for [XML configuration files](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#testcontext-ctx-management-xml).

| ![[Tip]](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/images/tip.png.pagespeed.ce.w22Wv-tZ37.png) |
| ---------------------------------------- |
| Support for using Groovy scripts to load an `ApplicationContext` in the Spring TestContext Framework is enabled automatically if Groovy is on the classpath. |

```
@RunWith(SpringRunner.class)
// ApplicationContext will be loaded from "/AppConfig.groovy" and
// "/TestConfig.groovy" in the root of the classpath
@ContextConfiguration({"/AppConfig.groovy", "/TestConfig.Groovy"})
public class MyTest {
	// class body...
}
```

If you omit both the `locations` and `value` attributes from the `@ContextConfiguration` annotation, the TestContext framework will attempt to detect a default Groovy script. Specifically, `GenericGroovyXmlContextLoader` and `GenericGroovyXmlWebContextLoader` detect a default location based on the name of the test class. If your class is named `com.example.MyTest`, the Groovy context loader will load your application context from `"classpath:com/example/MyTestContext.groovy"`.

```
package com.example;

@RunWith(SpringRunner.class)
// ApplicationContext will be loaded from
// "classpath:com/example/MyTestContext.groovy"
@ContextConfiguration
public class MyTest {
	// class body...
}
```

| ![[Tip]](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/images/tip.png.pagespeed.ce.w22Wv-tZ37.png) |
| ---------------------------------------- |
| Both XML configuration files and Groovy scripts can be declared simultaneously via the `locations` or `value` attribute of `@ContextConfiguration`. If the path to a configured resource location ends with `.xml` it will be loaded using an `XmlBeanDefinitionReader`; otherwise it will be loaded using a `GroovyBeanDefinitionReader`.The following listing demonstrates how to combine both in an integration test.`*@RunWith(SpringRunner.class)*// ApplicationContext will be loaded from// "/app-config.xml" and "/TestConfig.groovy"*@ContextConfiguration({ "/app-config.xml", "/TestConfig.groovy" })*public class MyTest {	// class body...}` |

#### Context configuration with annotated classes

To load an `ApplicationContext` for your tests using *annotated classes* (see [Section 3.12, “Java-based container configuration”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#beans-java)), annotate your test class with `@ContextConfiguration` and configure the `classes` attribute with an array that contains references to annotated classes.

```
@RunWith(SpringRunner.class)
// ApplicationContext will be loaded from AppConfig and TestConfig
@ContextConfiguration(classes = {AppConfig.class, TestConfig.class})
public class MyTest {
	// class body...
}
```

| ![[Tip]](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/images/tip.png.pagespeed.ce.w22Wv-tZ37.png) |
| ---------------------------------------- |
| The term *annotated class* can refer to any of the following.A class annotated with `@Configuration`A component (i.e., a class annotated with `@Component`, `@Service`, `@Repository`, etc.)A JSR-330 compliant class that is annotated with `javax.inject` annotationsAny other class that contains `@Bean`-methodsConsult the javadocs of `@Configuration` and `@Bean` for further information regarding the configuration and semantics of *annotated classes*, paying special attention to the discussion of *`@Bean` Lite Mode*. |

If you omit the `classes` attribute from the `@ContextConfiguration` annotation, the TestContext framework will attempt to detect the presence of default configuration classes. Specifically, `AnnotationConfigContextLoader` and `AnnotationConfigWebContextLoader` will detect all `static` nested classes of the test class that meet the requirements for configuration class implementations as specified in the `@Configuration` javadocs. In the following example, the `OrderServiceTest` class declares a `static` nested configuration class named `Config` that will be automatically used to load the `ApplicationContext` for the test class. Note that the name of the configuration class is arbitrary. In addition, a test class can contain more than one `static` nested configuration class if desired.

```
@RunWith(SpringRunner.class)
// ApplicationContext will be loaded from the
// static nested Config class
@ContextConfiguration
public class OrderServiceTest {

	@Configuration
	static class Config {

		// this bean will be injected into the OrderServiceTest class
		@Bean
		public OrderService orderService() {
			OrderService orderService = new OrderServiceImpl();
			// set properties, etc.
			return orderService;
		}
	}

	@Autowired
	private OrderService orderService;

	@Test
	public void testOrderService() {
		// test the orderService
	}

}
```

#### Mixing XML, Groovy scripts, and annotated classes

It may sometimes be desirable to mix XML configuration files, Groovy scripts, and annotated classes (i.e., typically `@Configuration` classes) to configure an`ApplicationContext` for your tests. For example, if you use XML configuration in production, you may decide that you want to use `@Configuration` classes to configure specific Spring-managed components for your tests, or vice versa.

Furthermore, some third-party frameworks (like Spring Boot) provide first-class support for loading an `ApplicationContext` from different types of resources simultaneously (e.g., XML configuration files, Groovy scripts, and `@Configuration` classes). The Spring Framework historically has not supported this for standard deployments. Consequently, most of the `SmartContextLoader` implementations that the Spring Framework delivers in the `spring-test` module support only one resource type per test context; however, this does not mean that you cannot use both. One exception to the general rule is that the `GenericGroovyXmlContextLoader`and `GenericGroovyXmlWebContextLoader` support both XML configuration files and Groovy scripts simultaneously. Furthermore, third-party frameworks may choose to support the declaration of both `locations` and `classes` via `@ContextConfiguration`, and with the standard testing support in the TestContext framework, you have the following options.

If you want to use resource locations (e.g., XML or Groovy) *and* `@Configuration` classes to configure your tests, you will have to pick one as the *entry point*, and that one will have to include or import the other. For example, in XML or Groovy scripts you can include `@Configuration` classes via component scanning or define them as normal Spring beans; whereas, in a `@Configuration` class you can use `@ImportResource` to import XML configuration files or Groovy scripts. Note that this behavior is semantically equivalent to how you configure your application in production: in production configuration you will define either a set of XML or Groovy resource locations or a set of `@Configuration` classes that your production `ApplicationContext` will be loaded from, but you still have the freedom to include or import the other type of configuration.

#### Context configuration with context initializers

To configure an `ApplicationContext` for your tests using context initializers, annotate your test class with `@ContextConfiguration` and configure the `initializers` attribute with an array that contains references to classes that implement `ApplicationContextInitializer`. The declared context initializers will then be used to initialize the `ConfigurableApplicationContext` that is loaded for your tests. Note that the concrete `ConfigurableApplicationContext` type supported by each declared initializer must be compatible with the type of `ApplicationContext` created by the `SmartContextLoader` in use (i.e., typically a `GenericApplicationContext`). Furthermore, the order in which the initializers are invoked depends on whether they implement Spring’s `Ordered` interface or are annotated with Spring’s `@Order` annotation or the standard `@Priority` annotation.

```
@RunWith(SpringRunner.class)
// ApplicationContext will be loaded from TestConfig
// and initialized by TestAppCtxInitializer
@ContextConfiguration(
	classes = TestConfig.class,
	initializers = TestAppCtxInitializer.class)
public class MyTest {
	// class body...
}
```

It is also possible to omit the declaration of XML configuration files, Groovy scripts, or annotated classes in `@ContextConfiguration` entirely and instead declare only`ApplicationContextInitializer` classes which are then responsible for registering beans in the context — for example, by programmatically loading bean definitions from XML files or configuration classes.

```
@RunWith(SpringRunner.class)
// ApplicationContext will be initialized by EntireAppInitializer
// which presumably registers beans in the context
@ContextConfiguration(initializers = EntireAppInitializer.class)
public class MyTest {
	// class body...
}
```

#### Context configuration inheritance

`@ContextConfiguration` supports boolean `inheritLocations` and `inheritInitializers` attributes that denote whether resource locations or annotated classes and context initializers declared by superclasses should be *inherited*. The default value for both flags is `true`. This means that a test class inherits the resource locations or annotated classes as well as the context initializers declared by any superclasses. Specifically, the resource locations or annotated classes for a test class are appended to the list of resource locations or annotated classes declared by superclasses. Similarly, the initializers for a given test class will be added to the set of initializers defined by test superclasses. Thus, subclasses have the option of *extending* the resource locations, annotated classes, or context initializers.

If the `inheritLocations` or `inheritInitializers` attribute in `@ContextConfiguration` is set to `false`, the resource locations or annotated classes and the context initializers, respectively, for the test class *shadow* and effectively replace the configuration defined by superclasses.

In the following example that uses XML resource locations, the `ApplicationContext` for `ExtendedTest` will be loaded from *"base-config.xml"* *and* *"extended-config.xml"*, in that order. Beans defined in *"extended-config.xml"* may therefore *override* (i.e., replace) those defined in *"base-config.xml"*.

```
@RunWith(SpringRunner.class)
// ApplicationContext will be loaded from "/base-config.xml"
// in the root of the classpath
@ContextConfiguration("/base-config.xml")
public class BaseTest {
	// class body...
}

// ApplicationContext will be loaded from "/base-config.xml" and
// "/extended-config.xml" in the root of the classpath
@ContextConfiguration("/extended-config.xml")
public class ExtendedTest extends BaseTest {
	// class body...
}
```

Similarly, in the following example that uses annotated classes, the `ApplicationContext` for `ExtendedTest` will be loaded from the `BaseConfig` *and*`ExtendedConfig` classes, in that order. Beans defined in `ExtendedConfig` may therefore override (i.e., replace) those defined in `BaseConfig`.

```
@RunWith(SpringRunner.class)
// ApplicationContext will be loaded from BaseConfig
@ContextConfiguration(classes = BaseConfig.class)
public class BaseTest {
	// class body...
}

// ApplicationContext will be loaded from BaseConfig and ExtendedConfig
@ContextConfiguration(classes = ExtendedConfig.class)
public class ExtendedTest extends BaseTest {
	// class body...
}
```

In the following example that uses context initializers, the `ApplicationContext` for `ExtendedTest` will be initialized using `BaseInitializer` *and*`ExtendedInitializer`. Note, however, that the order in which the initializers are invoked depends on whether they implement Spring’s `Ordered` interface or are annotated with Spring’s `@Order` annotation or the standard `@Priority` annotation.

```
@RunWith(SpringRunner.class)
// ApplicationContext will be initialized by BaseInitializer
@ContextConfiguration(initializers = BaseInitializer.class)
public class BaseTest {
	// class body...
}

// ApplicationContext will be initialized by BaseInitializer
// and ExtendedInitializer
@ContextConfiguration(initializers = ExtendedInitializer.class)
public class ExtendedTest extends BaseTest {
	// class body...
}
```

#### Context configuration with environment profiles

Spring 3.1 introduced first-class support in the framework for the notion of environments and profiles (a.k.a., *bean definition profiles*), and integration tests can be configured to activate particular bean definition profiles for various testing scenarios. This is achieved by annotating a test class with the `@ActiveProfiles` annotation and supplying a list of profiles that should be activated when loading the `ApplicationContext` for the test.

| ![[Note]](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| ---------------------------------------- |
| `@ActiveProfiles` may be used with any implementation of the new `SmartContextLoader` SPI, but `@ActiveProfiles` is not supported with implementations of the older `ContextLoader` SPI. |

Let’s take a look at some examples with XML configuration and `@Configuration` classes.

```
<!-- app-config.xml -->
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:jdbc="http://www.springframework.org/schema/jdbc"
	xmlns:jee="http://www.springframework.org/schema/jee"
	xsi:schemaLocation="...">

	<bean id="transferService"
			class="com.bank.service.internal.DefaultTransferService">
		<constructor-arg ref="accountRepository"/>
		<constructor-arg ref="feePolicy"/>
	</bean>

	<bean id="accountRepository"
			class="com.bank.repository.internal.JdbcAccountRepository">
		<constructor-arg ref="dataSource"/>
	</bean>

	<bean id="feePolicy"
		class="com.bank.service.internal.ZeroFeePolicy"/>

	<beans profile="dev">
		<jdbc:embedded-database id="dataSource">
			<jdbc:script
				location="classpath:com/bank/config/sql/schema.sql"/>
			<jdbc:script
				location="classpath:com/bank/config/sql/test-data.sql"/>
		</jdbc:embedded-database>
	</beans>

	<beans profile="production">
		<jee:jndi-lookup id="dataSource" jndi-name="java:comp/env/jdbc/datasource"/>
	</beans>

	<beans profile="default">
		<jdbc:embedded-database id="dataSource">
			<jdbc:script
				location="classpath:com/bank/config/sql/schema.sql"/>
		</jdbc:embedded-database>
	</beans>

</beans>
```

```
package com.bank.service;

@RunWith(SpringRunner.class)
// ApplicationContext will be loaded from "classpath:/app-config.xml"
@ContextConfiguration("/app-config.xml")
@ActiveProfiles("dev")
public class TransferServiceTest {

	@Autowired
	private TransferService transferService;

	@Test
	public void testTransferService() {
		// test the transferService
	}
}
```

When `TransferServiceTest` is run, its `ApplicationContext` will be loaded from the `app-config.xml` configuration file in the root of the classpath. If you inspect`app-config.xml` you’ll notice that the `accountRepository` bean has a dependency on a `dataSource` bean; however, `dataSource` is not defined as a top-level bean. Instead, `dataSource` is defined three times: in the *production* profile, the *dev* profile, and the *default* profile.

By annotating `TransferServiceTest` with `@ActiveProfiles("dev")` we instruct the Spring TestContext Framework to load the `ApplicationContext` with the active profiles set to `{"dev"}`. As a result, an embedded database will be created and populated with test data, and the `accountRepository` bean will be wired with a reference to the development `DataSource`. And that’s likely what we want in an integration test.

It is sometimes useful to assign beans to a `default` profile. Beans within the default profile are only included when no other profile is specifically activated. This can be used to define *fallback* beans to be used in the application’s default state. For example, you may explicitly provide a data source for `dev` and `production` profiles, but define an in-memory data source as a default when neither of these is active.

The following code listings demonstrate how to implement the same configuration and integration test but using `@Configuration` classes instead of XML.

```
@Configuration
@Profile("dev")
public class StandaloneDataConfig {

	@Bean
	public DataSource dataSource() {
		return new EmbeddedDatabaseBuilder()
			.setType(EmbeddedDatabaseType.HSQL)
			.addScript("classpath:com/bank/config/sql/schema.sql")
			.addScript("classpath:com/bank/config/sql/test-data.sql")
			.build();
	}
}
```

```
@Configuration
@Profile("production")
public class JndiDataConfig {

	@Bean(destroyMethod="")
	public DataSource dataSource() throws Exception {
		Context ctx = new InitialContext();
		return (DataSource) ctx.lookup("java:comp/env/jdbc/datasource");
	}
}
```

```
@Configuration
@Profile("default")
public class DefaultDataConfig {

	@Bean
	public DataSource dataSource() {
		return new EmbeddedDatabaseBuilder()
			.setType(EmbeddedDatabaseType.HSQL)
			.addScript("classpath:com/bank/config/sql/schema.sql")
			.build();
	}
}
```

```
@Configuration
public class TransferServiceConfig {

	@Autowired DataSource dataSource;

	@Bean
	public TransferService transferService() {
		return new DefaultTransferService(accountRepository(), feePolicy());
	}

	@Bean
	public AccountRepository accountRepository() {
		return new JdbcAccountRepository(dataSource);
	}

	@Bean
	public FeePolicy feePolicy() {
		return new ZeroFeePolicy();
	}

}
```

```
package com.bank.service;

@RunWith(SpringRunner.class)
@ContextConfiguration(classes = {
		TransferServiceConfig.class,
		StandaloneDataConfig.class,
		JndiDataConfig.class,
		DefaultDataConfig.class})
@ActiveProfiles("dev")
public class TransferServiceTest {

	@Autowired
	private TransferService transferService;

	@Test
	public void testTransferService() {
		// test the transferService
	}
}
```

In this variation, we have split the XML configuration into four independent `@Configuration` classes:

- `TransferServiceConfig`: acquires a `dataSource` via dependency injection using `@Autowired`
- `StandaloneDataConfig`: defines a `dataSource` for an embedded database suitable for developer tests
- `JndiDataConfig`: defines a `dataSource` that is retrieved from JNDI in a production environment
- `DefaultDataConfig`: defines a `dataSource` for a default embedded database in case no profile is active

As with the XML-based configuration example, we still annotate `TransferServiceTest` with `@ActiveProfiles("dev")`, but this time we specify all four configuration classes via the `@ContextConfiguration` annotation. The body of the test class itself remains completely unchanged.

It is often the case that a single set of profiles is used across multiple test classes within a given project. Thus, to avoid duplicate declarations of the `@ActiveProfiles`annotation it is possible to declare `@ActiveProfiles` once on a base class, and subclasses will automatically inherit the `@ActiveProfiles` configuration from the base class. In the following example, the declaration of `@ActiveProfiles` (as well as other annotations) has been moved to an abstract superclass, `AbstractIntegrationTest`.

```
package com.bank.service;

@RunWith(SpringRunner.class)
@ContextConfiguration(classes = {
		TransferServiceConfig.class,
		StandaloneDataConfig.class,
		JndiDataConfig.class,
		DefaultDataConfig.class})
@ActiveProfiles("dev")
public abstract class AbstractIntegrationTest {
}
```

```
package com.bank.service;

// "dev" profile inherited from superclass
public class TransferServiceTest extends AbstractIntegrationTest {

	@Autowired
	private TransferService transferService;

	@Test
	public void testTransferService() {
		// test the transferService
	}
}
```

`@ActiveProfiles` also supports an `inheritProfiles` attribute that can be used to disable the inheritance of active profiles.

```
package com.bank.service;

// "dev" profile overridden with "production"
@ActiveProfiles(profiles = "production", inheritProfiles = false)
public class ProductionTransferServiceTest extends AbstractIntegrationTest {
	// test body
}
```

Furthermore, it is sometimes necessary to resolve active profiles for tests *programmatically* instead of declaratively — for example, based on:

- the current operating system
- whether tests are being executed on a continuous integration build server
- the presence of certain environment variables
- the presence of custom class-level annotations
- etc.

To resolve active bean definition profiles programmatically, simply implement a custom `ActiveProfilesResolver` and register it via the `resolver` attribute of`@ActiveProfiles`. The following example demonstrates how to implement and register a custom `OperatingSystemActiveProfilesResolver`. For further information, refer to the corresponding javadocs.

```
package com.bank.service;

// "dev" profile overridden programmatically via a custom resolver
@ActiveProfiles(
	resolver = OperatingSystemActiveProfilesResolver.class,
	inheritProfiles = false)
public class TransferServiceTest extends AbstractIntegrationTest {
	// test body
}
```

```
package com.bank.service.test;

public class OperatingSystemActiveProfilesResolver implements ActiveProfilesResolver {

	@Override
	String[] resolve(Class<?> testClass) {
		String profile = ...;
		// determine the value of profile based on the operating system
		return new String[] {profile};
	}
}
```

#### Context configuration with test property sources

Spring 3.1 introduced first-class support in the framework for the notion of an environment with a hierarchy of *property sources*, and since Spring 4.1 integration tests can be configured with test-specific property sources. In contrast to the `@PropertySource` annotation used on `@Configuration` classes, the `@TestPropertySource`annotation can be declared on a test class to declare resource locations for test properties files or *inlined* properties. These test property sources will be added to the set of `PropertySources` in the `Environment` for the `ApplicationContext` loaded for the annotated integration test.

| ![[Note]](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| ---------------------------------------- |
| `@TestPropertySource` may be used with any implementation of the `SmartContextLoader` SPI, but `@TestPropertySource` is not supported with implementations of the older `ContextLoader` SPI.Implementations of `SmartContextLoader` gain access to merged test property source values via the `getPropertySourceLocations()` and `getPropertySourceProperties()` methods in `MergedContextConfiguration`. |

**Declaring test property sources**

Test properties files can be configured via the `locations` or `value` attribute of `@TestPropertySource` as shown in the following example.

Both traditional and XML-based properties file formats are supported — for example, `"classpath:/com/example/test.properties"` or `"file:///path/to/file.xml"`.

Each path will be interpreted as a Spring `Resource`. A plain path — for example, `"test.properties"` — will be treated as a classpath resource that is *relative* to the package in which the test class is defined. A path starting with a slash will be treated as an *absolute* classpath resource, for example: `"/org/example/test.xml"`. A path which references a URL (e.g., a path prefixed with `classpath:`, `file:`, `http:`, etc.) will be loaded using the specified resource protocol. Resource location wildcards (e.g. `***/**.properties`) are not permitted: each location must evaluate to exactly one `.properties` or `.xml` resource.

```
@ContextConfiguration
@TestPropertySource("/test.properties")
public class MyIntegrationTests {
	// class body...
}
```

*Inlined* properties in the form of key-value pairs can be configured via the `properties` attribute of `@TestPropertySource` as shown in the following example. All key-value pairs will be added to the enclosing `Environment` as a single test `PropertySource` with the highest precedence.

The supported syntax for key-value pairs is the same as the syntax defined for entries in a Java properties file:

- `"key=value"`
- `"key:value"`
- `"key value"`

```
@ContextConfiguration
@TestPropertySource(properties = {"timezone = GMT", "port: 4242"})
public class MyIntegrationTests {
	// class body...
}
```

**Default properties file detection**

If `@TestPropertySource` is declared as an empty annotation (i.e., without explicit values for the `locations` or `properties` attributes), an attempt will be made to detect a *default* properties file relative to the class that declared the annotation. For example, if the annotated test class is `com.example.MyTest`, the corresponding default properties file is `"classpath:com/example/MyTest.properties"`. If the default cannot be detected, an `IllegalStateException` will be thrown.

**Precedence**

Test property sources have higher precedence than those loaded from the operating system’s environment or Java system properties as well as property sources added by the application declaratively via `@PropertySource` or programmatically. Thus, test property sources can be used to selectively override properties defined in system and application property sources. Furthermore, inlined properties have higher precedence than properties loaded from resource locations.

In the following example, the `timezone` and `port` properties as well as any properties defined in `"/test.properties"` will override any properties of the same name that are defined in system and application property sources. Furthermore, if the `"/test.properties"` file defines entries for the `timezone` and `port` properties those will be overridden by the *inlined* properties declared via the `properties` attribute.

```
@ContextConfiguration
@TestPropertySource(
	locations = "/test.properties",
	properties = {"timezone = GMT", "port: 4242"}
)
public class MyIntegrationTests {
	// class body...
}
```

**Inheriting and overriding test property sources**

`@TestPropertySource` supports boolean `inheritLocations` and `inheritProperties` attributes that denote whether resource locations for properties files and inlined properties declared by superclasses should be *inherited*. The default value for both flags is `true`. This means that a test class inherits the locations and inlined properties declared by any superclasses. Specifically, the locations and inlined properties for a test class are appended to the locations and inlined properties declared by superclasses. Thus, subclasses have the option of *extending* the locations and inlined properties. Note that properties that appear later will *shadow* (i.e.., override) properties of the same name that appear earlier. In addition, the aforementioned precedence rules apply for inherited test property sources as well.

If the `inheritLocations` or `inheritProperties` attribute in `@TestPropertySource` is set to `false`, the locations or inlined properties, respectively, for the test class *shadow* and effectively replace the configuration defined by superclasses.

In the following example, the `ApplicationContext` for `BaseTest` will be loaded using only the `"base.properties"` file as a test property source. In contrast, the`ApplicationContext` for `ExtendedTest` will be loaded using the `"base.properties"` **and** `"extended.properties"` files as test property source locations.

```
@TestPropertySource("base.properties")
@ContextConfiguration
public class BaseTest {
	// ...
}

@TestPropertySource("extended.properties")
@ContextConfiguration
public class ExtendedTest extends BaseTest {
	// ...
}
```

In the following example, the `ApplicationContext` for `BaseTest` will be loaded using only the *inlined* `key1` property. In contrast, the `ApplicationContext` for `ExtendedTest` will be loaded using the *inlined* `key1` and `key2` properties.

```
@TestPropertySource(properties = "key1 = value1")
@ContextConfiguration
public class BaseTest {
	// ...
}

@TestPropertySource(properties = "key2 = value2")
@ContextConfiguration
public class ExtendedTest extends BaseTest {
	// ...
}
```

#### Loading a WebApplicationContext

Spring 3.2 introduced support for loading a `WebApplicationContext` in integration tests. To instruct the TestContext framework to load a `WebApplicationContext`instead of a standard `ApplicationContext`, simply annotate the respective test class with `@WebAppConfiguration`.

The presence of `@WebAppConfiguration` on your test class instructs the TestContext framework (TCF) that a `WebApplicationContext` (WAC) should be loaded for your integration tests. In the background the TCF makes sure that a `MockServletContext` is created and supplied to your test’s WAC. By default the base resource path for your `MockServletContext` will be set to *"src/main/webapp"*. This is interpreted as a path relative to the root of your JVM (i.e., normally the path to your project). If you’re familiar with the directory structure of a web application in a Maven project, you’ll know that *"src/main/webapp"* is the default location for the root of your WAR. If you need to override this default, simply provide an alternate path to the `@WebAppConfiguration` annotation (e.g., `@WebAppConfiguration("src/test/webapp")`). If you wish to reference a base resource path from the classpath instead of the file system, just use Spring’s *classpath:* prefix.

Please note that Spring’s testing support for `WebApplicationContexts` is on par with its support for standard `ApplicationContexts`. When testing with a `WebApplicationContext` you are free to declare XML configuration files, Groovy scripts, or `@Configuration` classes via `@ContextConfiguration`. You are of course also free to use any other test annotations such as `@ActiveProfiles`, `@TestExecutionListeners`, `@Sql`, `@Rollback`, etc.

The following examples demonstrate some of the various configuration options for loading a `WebApplicationContext`.

**Conventions. **

```
@RunWith(SpringRunner.class)

// defaults to "file:src/main/webapp"
@WebAppConfiguration

// detects "WacTests-context.xml" in same package
// or static nested @Configuration class
@ContextConfiguration

public class WacTests {
	//...
}
```

The above example demonstrates the TestContext framework’s support for *convention over configuration*. If you annotate a test class with `@WebAppConfiguration`without specifying a resource base path, the resource path will effectively default to *"file:src/main/webapp"*. Similarly, if you declare `@ContextConfiguration` without specifying resource `locations`, annotated `classes`, or context `initializers`, Spring will attempt to detect the presence of your configuration using conventions (i.e., *"WacTests-context.xml"* in the same package as the `WacTests` class or static nested `@Configuration` classes).

**Default resource semantics. **

```
@RunWith(SpringRunner.class)

// file system resource
@WebAppConfiguration("webapp")

// classpath resource
@ContextConfiguration("/spring/test-servlet-config.xml")

public class WacTests {
	//...
}
```

This example demonstrates how to explicitly declare a resource base path with `@WebAppConfiguration` and an XML resource location with `@ContextConfiguration`. The important thing to note here is the different semantics for paths with these two annotations. By default, `@WebAppConfiguration` resource paths are file system based; whereas, `@ContextConfiguration` resource locations are classpath based.

**Explicit resource semantics. **

```
@RunWith(SpringRunner.class)

// classpath resource
@WebAppConfiguration("classpath:test-web-resources")

// file system resource
@ContextConfiguration("file:src/main/webapp/WEB-INF/servlet-config.xml")

public class WacTests {
	//...
}
```

In this third example, we see that we can override the default resource semantics for both annotations by specifying a Spring resource prefix. Contrast the comments in this example with the previous example.

To provide comprehensive web testing support, Spring 3.2 introduced a `ServletTestExecutionListener` that is enabled by default. When testing against a`WebApplicationContext` this [TestExecutionListener](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#testcontext-key-abstractions) sets up default thread-local state via Spring Web’s `RequestContextHolder` before each test method and creates a `MockHttpServletRequest`, `MockHttpServletResponse`, and `ServletWebRequest` based on the base resource path configured via `@WebAppConfiguration`. `ServletTestExecutionListener` also ensures that the `MockHttpServletResponse` and `ServletWebRequest` can be injected into the test instance, and once the test is complete it cleans up thread-local state.

Once you have a `WebApplicationContext` loaded for your test you might find that you need to interact with the web mocks — for example, to set up your test fixture or to perform assertions after invoking your web component. The following example demonstrates which mocks can be autowired into your test instance. Note that the`WebApplicationContext` and `MockServletContext` are both cached across the test suite; whereas, the other mocks are managed per test method by the`ServletTestExecutionListener`.

**Injecting mocks. **

```
@WebAppConfiguration
@ContextConfiguration
public class WacTests {

	@Autowired
	WebApplicationContext wac; // cached

	@Autowired
	MockServletContext servletContext; // cached

	@Autowired
	MockHttpSession session;

	@Autowired
	MockHttpServletRequest request;

	@Autowired
	MockHttpServletResponse response;

	@Autowired
	ServletWebRequest webRequest;

	//...
}
```

#### Context caching

Once the TestContext framework loads an `ApplicationContext` (or `WebApplicationContext`) for a test, that context will be cached and reused for *all* subsequent tests that declare the same unique context configuration within the same test suite. To understand how caching works, it is important to understand what is meant by *unique* and *test suite*.

An `ApplicationContext` can be *uniquely* identified by the combination of configuration parameters that are used to load it. Consequently, the unique combination of configuration parameters are used to generate a *key* under which the context is cached. The TestContext framework uses the following configuration parameters to build the context cache key:

- `locations` *(from @ContextConfiguration)*
- `classes` *(from @ContextConfiguration)*
- `contextInitializerClasses` *(from @ContextConfiguration)*
- `contextCustomizers` *(from ContextCustomizerFactory)*
- `contextLoader` *(from @ContextConfiguration)*
- `parent` *(from @ContextHierarchy)*
- `activeProfiles` *(from @ActiveProfiles)*
- `propertySourceLocations` *(from @TestPropertySource)*
- `propertySourceProperties` *(from @TestPropertySource)*
- `resourceBasePath` *(from @WebAppConfiguration)*

For example, if `TestClassA` specifies `{"app-config.xml", "test-config.xml"}` for the `locations` (or `value`) attribute of `@ContextConfiguration`, the TestContext framework will load the corresponding `ApplicationContext` and store it in a `static` context cache under a key that is based solely on those locations. So if `TestClassB` also defines `{"app-config.xml", "test-config.xml"}` for its locations (either explicitly or implicitly through inheritance) but does not define `@WebAppConfiguration`, a different `ContextLoader`, different active profiles, different context initializers, different test property sources, or a different parent context, then the same `ApplicationContext` will be shared by both test classes. This means that the setup cost for loading an application context is incurred only once (per test suite), and subsequent test execution is much faster.

| ![[Note]](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| ---------------------------------------- |
| The Spring TestContext framework stores application contexts in a *static* cache. This means that the context is literally stored in a `static` variable. In other words, if tests execute in separate processes the static cache will be cleared between each test execution, and this will effectively disable the caching mechanism.To benefit from the caching mechanism, all tests must run within the same process or test suite. This can be achieved by executing all tests as a group within an IDE. Similarly, when executing tests with a build framework such as Ant, Maven, or Gradle it is important to make sure that the build framework does not *fork* between tests. For example, if the [forkMode](https://maven.apache.org/plugins/maven-surefire-plugin/test-mojo.html#forkMode) for the Maven Surefire plug-in is set to `always` or `pertest`, the TestContext framework will not be able to cache application contexts between test classes and the build process will run significantly slower as a result. |

Since Spring Framework 4.3, the size of the context cache is bounded with a default maximum size of 32. Whenever the maximum size is reached, a *least recently used*(LRU) eviction policy is used to evict and close stale contexts. The maximum size can be  configured from the command line or a build script by setting a JVM system property named `spring.test.context.cache.maxSize`. As an alternative, the same property can be set programmatically via the `SpringProperties` API.

Since having a large number of application contexts loaded within a given test suite can cause the suite to take an unnecessarily long time to execute, it is often beneficial to know exactly how many contexts have been loaded and cached. To view the statistics for the underlying context cache, simply set the log level for the`org.springframework.test.context.cache` logging category to `DEBUG`.

In the unlikely case that a test corrupts the application context and requires reloading — for example, by modifying a bean definition or the state of an application object — you can annotate your test class or test method with `@DirtiesContext` (see the discussion of `@DirtiesContext` in [Section 11.4.1, “Spring Testing Annotations”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#integration-testing-annotations-spring)). This instructs Spring to remove the context from the cache and rebuild the application context before executing the next test. Note that support for the `@DirtiesContext` annotation is provided by the `DirtiesContextBeforeModesTestExecutionListener` and the `DirtiesContextTestExecutionListener` which are enabled by default.

#### Context hierarchies

When writing integration tests that rely on a loaded Spring `ApplicationContext`, it is often sufficient to test against a single context; however, there are times when it is beneficial or even necessary to test against a hierarchy of `ApplicationContext`s. For example, if you are developing a Spring MVC web application you will typically have a root `WebApplicationContext` loaded via Spring’s `ContextLoaderListener` and a child `WebApplicationContext` loaded via Spring’s `DispatcherServlet`. This results in a parent-child context hierarchy where shared components and infrastructure configuration are declared in the root context and consumed in the child context by web-specific components. Another use case can be found in Spring Batch applications where you often have a parent context that provides configuration for shared batch infrastructure and a child context for the configuration of a specific batch job.

Since Spring Framework 3.2.2, it is possible to write integration tests that use context hierarchies by declaring context configuration via the `@ContextHierarchy`annotation, either on an individual test class or within a test class hierarchy. If a context hierarchy is declared on multiple classes within a test class hierarchy it is also possible to merge or override the context configuration for a specific, named level in the context hierarchy. When merging configuration for a given level in the hierarchy the configuration resource type (i.e., XML configuration files or annotated classes) must be consistent; otherwise, it is perfectly acceptable to have different levels in a context hierarchy configured using different resource types.

The following JUnit 4 based examples demonstrate common configuration scenarios for integration tests that require the use of context hierarchies.

`ControllerIntegrationTests` represents a typical integration testing scenario for a Spring MVC web application by declaring a context hierarchy consisting of two levels, one for the *root* WebApplicationContext (loaded using the `TestAppConfig` `@Configuration` class) and one for the *dispatcher servlet*`WebApplicationContext` (loaded using the `WebConfig` `@Configuration` class). The `WebApplicationContext` that is *autowired* into the test instance is the one for the child context (i.e., the lowest context in the hierarchy).

```
@RunWith(SpringRunner.class)
@WebAppConfiguration
@ContextHierarchy({
	@ContextConfiguration(classes = TestAppConfig.class),
	@ContextConfiguration(classes = WebConfig.class)
})
public class ControllerIntegrationTests {

	@Autowired
	private WebApplicationContext wac;

	// ...
}
```

The following test classes define a context hierarchy within a test class hierarchy. `AbstractWebTests` declares the configuration for a root `WebApplicationContext` in a Spring-powered web application. Note, however, that `AbstractWebTests` does not declare `@ContextHierarchy`; consequently, subclasses of `AbstractWebTests`can optionally participate in a context hierarchy or simply follow the standard semantics for `@ContextConfiguration`. `SoapWebServiceTests` and `RestWebServiceTests` both extend `AbstractWebTests` and define a context hierarchy via `@ContextHierarchy`. The result is that three application contexts will be loaded (one for each declaration of `@ContextConfiguration`), and the application context loaded based on the configuration in `AbstractWebTests` will be set as the parent context for each of the contexts loaded for the concrete subclasses.

```
@RunWith(SpringRunner.class)
@WebAppConfiguration
@ContextConfiguration("file:src/main/webapp/WEB-INF/applicationContext.xml")
public abstract class AbstractWebTests {}

@ContextHierarchy(@ContextConfiguration("/spring/soap-ws-config.xml")
public class SoapWebServiceTests extends AbstractWebTests {}

@ContextHierarchy(@ContextConfiguration("/spring/rest-ws-config.xml")
public class RestWebServiceTests extends AbstractWebTests {}
```

The following classes demonstrate the use of *named* hierarchy levels in order to *merge* the configuration for specific levels in a context hierarchy. `BaseTests` defines two levels in the hierarchy, `parent` and `child`. `ExtendedTests` extends `BaseTests` and instructs the Spring TestContext Framework to merge the context configuration for the `child` hierarchy level, simply by ensuring that the names declared via the `name` attribute in `@ContextConfiguration` are both `"child"`. The result is that three application contexts will be loaded: one for `"/app-config.xml"`, one for `"/user-config.xml"`, and one for `{"/user-config.xml", "/order-config.xml"}`. As with the previous example, the application context loaded from `"/app-config.xml"` will be set as the parent context for the contexts loaded from `"/user-config.xml"` and `{"/user-config.xml", "/order-config.xml"}`.

```
@RunWith(SpringRunner.class)
@ContextHierarchy({
	@ContextConfiguration(name = "parent", locations = "/app-config.xml"),
	@ContextConfiguration(name = "child", locations = "/user-config.xml")
})
public class BaseTests {}

@ContextHierarchy(
	@ContextConfiguration(name = "child", locations = "/order-config.xml")
)
public class ExtendedTests extends BaseTests {}
```

In contrast to the previous example, this example demonstrates how to *override* the configuration for a given named level in a context hierarchy by setting the`inheritLocations` flag in `@ContextConfiguration` to `false`. Consequently, the application context for `ExtendedTests` will be loaded only from`"/test-user-config.xml"` and will have its parent set to the context loaded from `"/app-config.xml"`.

```
@RunWith(SpringRunner.class)
@ContextHierarchy({
	@ContextConfiguration(name = "parent", locations = "/app-config.xml"),
	@ContextConfiguration(name = "child", locations = "/user-config.xml")
})
public class BaseTests {}

@ContextHierarchy(
	@ContextConfiguration(
		name = "child",
		locations = "/test-user-config.xml",
		inheritLocations = false
))
public class ExtendedTests extends BaseTests {}
```

| ![[Note]](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| ---------------------------------------- |
| If `@DirtiesContext` is used in a test whose context is configured as part of a context hierarchy, the `hierarchyMode` flag can be used to control how the context cache is cleared. For further details consult the discussion of `@DirtiesContext` in [Spring Testing Annotations](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#integration-testing-annotations-spring) and the `@DirtiesContext`javadocs. |

### 11.5.5 Dependency injection of test fixtures

When you use the `DependencyInjectionTestExecutionListener` — which is configured by default — the dependencies of your test instances are *injected* from beans in the application context that you configured with `@ContextConfiguration`. You may use setter injection, field injection, or both, depending on which annotations you choose and whether you place them on setter methods or fields. For consistency with the annotation support introduced in Spring 2.5 and 3.0, you can use Spring’s `@Autowired` annotation or the `@Inject` annotation from JSR 330.

| ![[Tip]](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/images/tip.png.pagespeed.ce.w22Wv-tZ37.png) |
| ---------------------------------------- |
| The TestContext framework does not instrument the manner in which a test instance is instantiated. Thus the use of `@Autowired` or `@Inject` for constructors has no effect for test classes. |

Because `@Autowired` is used to perform [*autowiring by type*](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#beans-factory-autowire), if you have multiple bean definitions of the same type, you cannot rely on this approach for those particular beans. In that case, you can use `@Autowired` in conjunction with `@Qualifier`. As of Spring 3.0 you may also choose to use `@Inject` in conjunction with `@Named`. Alternatively, if your test class has access to its `ApplicationContext`, you can perform an explicit lookup by using (for example) a call to `applicationContext.getBean("titleRepository")`.

If you do not want dependency injection applied to your test instances, simply do not annotate fields or setter methods with `@Autowired` or `@Inject`. Alternatively, you can disable dependency injection altogether by explicitly configuring your class with `@TestExecutionListeners` and omitting `DependencyInjectionTestExecutionListener.class` from the list of listeners.

Consider the scenario of testing a `HibernateTitleRepository` class, as outlined in the [Goals](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#integration-testing-goals) section. The next two code listings demonstrate the use of `@Autowired`on fields and setter methods. The application context configuration is presented after all sample code listings.

| ![[Note]](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| ---------------------------------------- |
| The dependency injection behavior in the following code listings is not specific to JUnit 4. The same DI techniques can be used in conjunction with any testing framework.The following examples make calls to static assertion methods such as `assertNotNull()` but without prepending the call with `Assert`. In such cases, assume that the method was properly imported through an `import static` declaration that is not shown in the example. |

The first code listing shows a JUnit 4 based implementation of the test class that uses `@Autowired` for field injection.

```
@RunWith(SpringRunner.class)
// specifies the Spring configuration to load for this test fixture
@ContextConfiguration("repository-config.xml")
public class HibernateTitleRepositoryTests {

	// this instance will be dependency injected by type
	@Autowired
	private HibernateTitleRepository titleRepository;

	@Test
	public void findById() {
		Title title = titleRepository.findById(new Long(10));
		assertNotNull(title);
	}
}
```

Alternatively, you can configure the class to use `@Autowired` for setter injection as seen below.

```
@RunWith(SpringRunner.class)
// specifies the Spring configuration to load for this test fixture
@ContextConfiguration("repository-config.xml")
public class HibernateTitleRepositoryTests {

	// this instance will be dependency injected by type
	private HibernateTitleRepository titleRepository;

	@Autowired
	public void setTitleRepository(HibernateTitleRepository titleRepository) {
		this.titleRepository = titleRepository;
	}

	@Test
	public void findById() {
		Title title = titleRepository.findById(new Long(10));
		assertNotNull(title);
	}
}
```

The preceding code listings use the same XML context file referenced by the `@ContextConfiguration` annotation (that is, `repository-config.xml`), which looks like this:

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
		http://www.springframework.org/schema/beans/spring-beans.xsd">

	<!-- this bean will be injected into the HibernateTitleRepositoryTests class -->
	<bean id="titleRepository" class="com.foo.repository.hibernate.HibernateTitleRepository">
		<property name="sessionFactory" ref="sessionFactory"/>
	</bean>

	<bean id="sessionFactory" class="org.springframework.orm.hibernate5.LocalSessionFactoryBean">
		<!-- configuration elided for brevity -->
	</bean>

</beans>
```

| ![[Note]](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| ---------------------------------------- |
| If you are extending from a Spring-provided test base class that happens to use `@Autowired` on one of its setter methods, you might have multiple beans of the affected type defined in your application context: for example, multiple `DataSource` beans. In such a case, you can override the setter method and use the `@Qualifier` annotation to indicate a specific target bean as follows, but make sure to delegate to the overridden method in the superclass as well.`// ...	*@Autowired*	*@Override*	public void setDataSource(**@Qualifier("myDataSource")** DataSource dataSource) {		**super**.setDataSource(dataSource);	}// ...`The specified qualifier value indicates the specific `DataSource` bean to inject, narrowing the set of type matches to a specific bean. Its value is matched against `<qualifier>` declarations within the corresponding `<bean>` definitions. The bean name is used as a fallback qualifier value, so you may effectively also point to a specific bean by name there (as shown above, assuming that "myDataSource" is the bean id). |

### 11.5.6 Testing request and session scoped beans

[Request and session scoped beans](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#beans-factory-scopes-other) have been supported by Spring since the early years, and since Spring 3.2 it’s a breeze to test your request-scoped and session-scoped beans by following these steps.

- Ensure that a `WebApplicationContext` is loaded for your test by annotating your test class with `@WebAppConfiguration`.
- Inject the mock request or session into your test instance and prepare your test fixture as appropriate.
- Invoke your web component that you retrieved from the configured `WebApplicationContext` (i.e., via dependency injection).
- Perform assertions against the mocks.

The following code snippet displays the XML configuration for a login use case. Note that the `userService` bean has a dependency on a request-scoped `loginAction`bean. Also, the `LoginAction` is instantiated using [SpEL expressions](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#expressions) that retrieve the username and password from the current HTTP request. In our test, we will want to configure these request parameters via the mock managed by the TestContext framework.

**Request-scoped bean configuration. **

```
<beans>

	<bean id="userService"
			class="com.example.SimpleUserService"
			c:loginAction-ref="loginAction" />

	<bean id="loginAction" class="com.example.LoginAction"
			c:username="{request.getParameter('user')}"
			c:password="{request.getParameter('pswd')}"
			scope="request">
		<aop:scoped-proxy />
	</bean>

</beans>
```

In `RequestScopedBeanTests` we inject both the `UserService` (i.e., the subject under test) and the `MockHttpServletRequest` into our test instance. Within our`requestScope()` test method we set up our test fixture by setting request parameters in the provided `MockHttpServletRequest`. When the `loginUser()` method is invoked on our `userService` we are assured that the user service has access to the request-scoped `loginAction` for the current `MockHttpServletRequest` (i.e., the one we just set parameters in). We can then perform assertions against the results based on the known inputs for the username and password.

**Request-scoped bean test. **

```
@RunWith(SpringRunner.class)
@ContextConfiguration
@WebAppConfiguration
public class RequestScopedBeanTests {

	@Autowired UserService userService;
	@Autowired MockHttpServletRequest request;

	@Test
	public void requestScope() {

		request.setParameter("user", "enigma");
		request.setParameter("pswd", "$pr!ng");

		LoginResults results = userService.loginUser();

		// assert results
	}
}
```

The following code snippet is similar to the one we saw above for a request-scoped bean; however, this time the `userService` bean has a dependency on a session-scoped `userPreferences` bean. Note that the `UserPreferences` bean is instantiated using a SpEL expression that retrieves the *theme* from the current HTTP session. In our test, we will need to configure a theme in the mock session managed by the TestContext framework.

**Session-scoped bean configuration. **

```
<beans>

	<bean id="userService"
			class="com.example.SimpleUserService"
			c:userPreferences-ref="userPreferences" />

	<bean id="userPreferences"
			class="com.example.UserPreferences"
			c:theme="#{session.getAttribute('theme')}"
			scope="session">
		<aop:scoped-proxy />
	</bean>

</beans>
```

In `SessionScopedBeanTests` we inject the `UserService` and the `MockHttpSession` into our test instance. Within our `sessionScope()` test method we set up our test fixture by setting the expected "theme" attribute in the provided `MockHttpSession`. When the `processUserPreferences()` method is invoked on our `userService` we are assured that the user service has access to the session-scoped `userPreferences` for the current `MockHttpSession`, and we can perform assertions against the results based on the configured theme.

**Session-scoped bean test. **

```
@RunWith(SpringRunner.class)
@ContextConfiguration
@WebAppConfiguration
public class SessionScopedBeanTests {

	@Autowired UserService userService;
	@Autowired MockHttpSession session;

	@Test
	public void sessionScope() throws Exception {

		session.setAttribute("theme", "blue");

		Results results = userService.processUserPreferences();

		// assert results
	}
}
```

### 11.5.7 Transaction management

In the TestContext framework, transactions are managed by the `TransactionalTestExecutionListener` which is configured by default, even if you do not explicitly declare `@TestExecutionListeners` on your test class. To enable support for transactions, however, you must configure a `PlatformTransactionManager` bean in the`ApplicationContext` that is loaded via `@ContextConfiguration` semantics (further details are provided below). In addition, you must declare Spring’s `@Transactional` annotation either at the class or method level for your tests.

#### Test-managed transactions

*Test-managed transactions* are transactions that are managed *declaratively* via the `TransactionalTestExecutionListener` or *programmatically* via `TestTransaction` (see below). Such transactions should not be confused with *Spring-managed transactions* (i.e., those managed directly by Spring within the `ApplicationContext` loaded for tests) or *application-managed transactions* (i.e., those managed programmatically within application code that is invoked via tests). Spring-managed and application-managed transactions will typically participate in test-managed transactions; however, caution should be taken if Spring-managed or application-managed transactions are configured with any *propagation* type other than `REQUIRED` or `SUPPORTS` (see the discussion on [transaction propagation](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#tx-propagation) for details).

#### Enabling and disabling transactions

Annotating a test method with `@Transactional` causes the test to be run within a transaction that will, by default, be automatically rolled back after completion of the test. If a test class is annotated with `@Transactional`, each test method within that class hierarchy will be run within a transaction. Test methods that are not annotated with `@Transactional` (at the class or method level) will not be run within a transaction. Furthermore, tests that are annotated with `@Transactional` but have the`propagation` type set to `NOT_SUPPORTED` will not be run within a transaction.

*Note that AbstractTransactionalJUnit4SpringContextTests and AbstractTransactionalTestNGSpringContextTests are preconfigured for transactional support at the class level.*

The following example demonstrates a common scenario for writing an integration test for a Hibernate-based `UserRepository`. As explained in [the section called “Transaction rollback and commit behavior”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#testcontext-tx-rollback-and-commit-behavior), there is no need to clean up the database after the `createUser()` method is executed since any changes made to the database will be automatically rolled back by the `TransactionalTestExecutionListener`. See [Section 11.7, “PetClinic Example”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#testing-examples-petclinic) for an additional example.

```
@RunWith(SpringRunner.class)
@ContextConfiguration(classes = TestConfig.class)
@Transactional
public class HibernateUserRepositoryTests {

	@Autowired
	HibernateUserRepository repository;

	@Autowired
	SessionFactory sessionFactory;

	JdbcTemplate jdbcTemplate;

	@Autowired
	public void setDataSource(DataSource dataSource) {
		this.jdbcTemplate = new JdbcTemplate(dataSource);
	}

	@Test
	public void createUser() {
		// track initial state in test database:
		final int count = countRowsInTable("user");

		User user = new User(...);
		repository.save(user);

		// Manual flush is required to avoid false positive in test
		sessionFactory.getCurrentSession().flush();
		assertNumUsers(count + 1);
	}

	protected int countRowsInTable(String tableName) {
		return JdbcTestUtils.countRowsInTable(this.jdbcTemplate, tableName);
	}

	protected void assertNumUsers(int expected) {
		assertEquals("Number of rows in the [user] table.", expected, countRowsInTable("user"));
	}
}
```

#### Transaction rollback and commit behavior

By default, test transactions will be automatically rolled back after completion of the test; however, transactional commit and rollback behavior can be configured declaratively via the `@Commit` and `@Rollback` annotations. See the corresponding entries in the [annotation support](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#integration-testing-annotations) section for further details.

#### Programmatic transaction management

Since Spring Framework 4.1, it is possible to interact with test-managed transactions *programmatically* via the static methods in `TestTransaction`. For example,`TestTransaction` may be used within *test* methods, *before* methods, and *after* methods to start or end the current test-managed transaction or to configure the current test-managed transaction for rollback or commit. Support for `TestTransaction` is automatically available whenever the `TransactionalTestExecutionListener` is enabled.

The following example demonstrates some of the features of `TestTransaction`. Consult the javadocs for `TestTransaction` for further details.

```
@ContextConfiguration(classes = TestConfig.class)
public class ProgrammaticTransactionManagementTests extends
		AbstractTransactionalJUnit4SpringContextTests {

	@Test
	public void transactionalTest() {
		// assert initial state in test database:
		assertNumUsers(2);

		deleteFromTables("user");

		// changes to the database will be committed!
		TestTransaction.flagForCommit();
		TestTransaction.end();
		assertFalse(TestTransaction.isActive());
		assertNumUsers(0);

		TestTransaction.start();
		// perform other actions against the database that will
		// be automatically rolled back after the test completes...
	}

	protected void assertNumUsers(int expected) {
		assertEquals("Number of rows in the [user] table.", expected, countRowsInTable("user"));
	}
}
```

#### Executing code outside of a transaction

Occasionally you need to execute certain code before or after a transactional test method but outside the transactional context — for example, to verify the initial database state prior to execution of your test or to verify expected transactional commit behavior after test execution (if the test was configured to commit the transaction). `TransactionalTestExecutionListener` supports the `@BeforeTransaction` and `@AfterTransaction` annotations exactly for such scenarios. Simply annotate any `void` method in a test class or any `void` default method in a test interface with one of these annotations, and the `TransactionalTestExecutionListener` ensures that your *before transaction method* or *after transaction method* is executed at the appropriate time.

| ![[Tip]](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/images/tip.png.pagespeed.ce.w22Wv-tZ37.png) |
| ---------------------------------------- |
| Any *before methods* (such as methods annotated with JUnit 4’s `@Before`) and any *after methods* (such as methods annotated with JUnit 4’s `@After`) are executed *within* a transaction. In addition, methods annotated with `@BeforeTransaction` or `@AfterTransaction` are naturally not executed for test methods that are not configured to run within a transaction. |

#### Configuring a transaction manager

`TransactionalTestExecutionListener` expects a `PlatformTransactionManager` bean to be defined in the Spring `ApplicationContext` for the test. In case there are multiple instances of `PlatformTransactionManager` within the test’s `ApplicationContext`, a *qualifier* may be declared via `@Transactional("myTxMgr")`or `@Transactional(transactionManager = "myTxMgr")`, or `TransactionManagementConfigurer` can be implemented by an `@Configuration` class. Consult the javadocs for `TestContextTransactionUtils.retrieveTransactionManager()` for details on the algorithm used to look up a transaction manager in the test’s `ApplicationContext`.

#### Demonstration of all transaction-related annotations

The following JUnit 4 based example displays a fictitious integration testing scenario highlighting all transaction-related annotations. The example is **not** intended to demonstrate best practices but rather to demonstrate how these annotations can be used. Consult the [annotation support](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#integration-testing-annotations) section for further information and configuration examples. [Transaction management for `@Sql`](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#testcontext-executing-sql-declaratively-tx) contains an additional example using `@Sql` for declarative SQL script execution with default transaction rollback semantics.

```
@RunWith(SpringRunner.class)
@ContextConfiguration
@Transactional(transactionManager = "txMgr")
@Commit
public class FictitiousTransactionalTest {

	@BeforeTransaction
	void verifyInitialDatabaseState() {
		// logic to verify the initial state before a transaction is started
	}

	@Before
	public void setUpTestDataWithinTransaction() {
		// set up test data within the transaction
	}

	@Test
	// overrides the class-level @Commit setting
	@Rollback
	public void modifyDatabaseWithinTransaction() {
		// logic which uses the test data and modifies database state
	}

	@After
	public void tearDownWithinTransaction() {
		// execute "tear down" logic within the transaction
	}

	@AfterTransaction
	void verifyFinalDatabaseState() {
		// logic to verify the final state after transaction has rolled back
	}

}
```

| ![[Note]](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| ---------------------------------------- |
| When you test application code that manipulates the state of a Hibernate session or JPA persistence context, make sure to *flush* the underlying unit of work within test methods that execute that code. Failing to flush the underlying unit of work can produce *false positives*: your test may pass, but the same code throws an exception in a live, production environment. In the following Hibernate-based example test case, one method demonstrates a false positive, and the other method correctly exposes the results of flushing the session. Note that this applies to any ORM frameworks that maintain an in-memory *unit of work*.`// ...*@Autowired*SessionFactory sessionFactory;*@Transactional**@Test* // no expected exception!public void falsePositive() {	updateEntityInHibernateSession();	// False positive: an exception will be thrown once the Hibernate	// Session is finally flushed (i.e., in production code)}*@Transactional**@Test(expected = ...)*public void updateWithSessionFlush() {	updateEntityInHibernateSession();	// Manual flush is required to avoid false positive in test	sessionFactory.getCurrentSession().flush();}// ...`Or for JPA:`// ...*@PersistenceContext*EntityManager entityManager;*@Transactional**@Test* // no expected exception!public void falsePositive() {	updateEntityInJpaPersistenceContext();	// False positive: an exception will be thrown once the JPA	// EntityManager is finally flushed (i.e., in production code)}*@Transactional**@Test(expected = ...)*public void updateWithEntityManagerFlush() {	updateEntityInJpaPersistenceContext();	// Manual flush is required to avoid false positive in test	entityManager.flush();}// ...` |

### 11.5.8 Executing SQL scripts

When writing integration tests against a relational database, it is often beneficial to execute SQL scripts to modify the database schema or insert test data into tables. The `spring-jdbc` module provides support for *initializing* an embedded or existing database by executing SQL scripts when the Spring `ApplicationContext` is loaded. See [Section 15.8, “Embedded database support”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#jdbc-embedded-database-support) and [Section 15.8.5, “Testing data access logic with an embedded database”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#jdbc-embedded-database-dao-testing) for details.

Although it is very useful to initialize a database for testing *once* when the `ApplicationContext` is loaded, sometimes it is essential to be able to modify the database *during* integration tests. The following sections explain how to execute SQL scripts programmatically and declaratively during integration tests.

#### Executing SQL scripts programmatically

Spring provides the following options for executing SQL scripts programmatically within integration test methods.

- `org.springframework.jdbc.datasource.init.ScriptUtils`
- `org.springframework.jdbc.datasource.init.ResourceDatabasePopulator`
- `org.springframework.test.context.junit4.AbstractTransactionalJUnit4SpringContextTests`
- `org.springframework.test.context.testng.AbstractTransactionalTestNGSpringContextTests`

`ScriptUtils` provides a collection of static utility methods for working with SQL scripts and is mainly intended for internal use within the framework. However, if you require full control over how SQL scripts are parsed and executed, `ScriptUtils` may suit your needs better than some of the other alternatives described below. Consult the javadocs for individual methods in `ScriptUtils` for further details.

`ResourceDatabasePopulator` provides a simple object-based API for programmatically populating, initializing, or cleaning up a database using SQL scripts defined in external resources. `ResourceDatabasePopulator` provides options for configuring the character encoding, statement separator, comment delimiters, and error handling flags used when parsing and executing the scripts, and each of the configuration options has a reasonable default value. Consult the javadocs for details on default values. To execute the scripts configured in a `ResourceDatabasePopulator`, you can invoke either the `populate(Connection)` method to execute the populator against a `java.sql.Connection` or the `execute(DataSource)` method to execute the populator against a `javax.sql.DataSource`. The following example specifies SQL scripts for a test schema and test data, sets the statement separator to `"@@"`, and then executes the scripts against a `DataSource`.

```
@Test
public void databaseTest {
	ResourceDatabasePopulator populator = new ResourceDatabasePopulator();
	populator.addScripts(
		new ClassPathResource("test-schema.sql"),
		new ClassPathResource("test-data.sql"));
	populator.setSeparator("@@");
	populator.execute(this.dataSource);
	// execute code that uses the test schema and data
}
```

Note that `ResourceDatabasePopulator` internally delegates to `ScriptUtils` for parsing and executing SQL scripts. Similarly, the `executeSqlScript(..)` methods in [`AbstractTransactionalJUnit4SpringContextTests`](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#testcontext-support-classes-junit4) and [`AbstractTransactionalTestNGSpringContextTests`](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#testcontext-support-classes-testng) internally use a `ResourceDatabasePopulator` for executing SQL scripts. Consult the javadocs for the various `executeSqlScript(..)` methods for further details.

#### Executing SQL scripts declaratively with @Sql

In addition to the aforementioned mechanisms for executing SQL scripts *programmatically*, SQL scripts can also be configured *declaratively* in the Spring TestContext Framework. Specifically, the `@Sql` annotation can be declared on a test class or test method to configure the resource paths to SQL scripts that should be executed against a given database either before or after an integration test method. Note that method-level declarations override class-level declarations and that support for`@Sql` is provided by the `SqlScriptsTestExecutionListener` which is enabled by default.

**Path resource semantics**

Each path will be interpreted as a Spring `Resource`. A plain path — for example, `"schema.sql"` — will be treated as a classpath resource that is *relative* to the package in which the test class is defined. A path starting with a slash will be treated as an *absolute* classpath resource, for example: `"/org/example/schema.sql"`. A path which references a URL (e.g., a path prefixed with `classpath:`, `file:`, `http:`, etc.) will be loaded using the specified resource protocol.

The following example demonstrates how to use `@Sql` at the class level and at the method level within a JUnit 4 based integration test class.

```
@RunWith(SpringRunner.class)
@ContextConfiguration
@Sql("/test-schema.sql")
public class DatabaseTests {

	@Test
	public void emptySchemaTest {
		// execute code that uses the test schema without any test data
	}

	@Test
	@Sql({"/test-schema.sql", "/test-user-data.sql"})
	public void userTest {
		// execute code that uses the test schema and test data
	}
}
```

**Default script detection**

If no SQL scripts are specified, an attempt will be made to detect a `default` script depending on where `@Sql` is declared. If a default cannot be detected, an`IllegalStateException` will be thrown.

- *class-level declaration*: if the annotated test class is `com.example.MyTest`, the corresponding default script is `"classpath:com/example/MyTest.sql"`.
- *method-level declaration*: if the annotated test method is named `testMethod()` and is defined in the class `com.example.MyTest`, the corresponding default script is `"classpath:com/example/MyTest.testMethod.sql"`.

**Declaring multiple @Sql sets**

If multiple sets of SQL scripts need to be configured for a given test class or test method but with different syntax configuration, different error handling rules, or different execution phases per set, it is possible to declare multiple instances of `@Sql`. With Java 8, `@Sql` can be used as a *repeatable* annotation. Otherwise, the `@SqlGroup`annotation can be used as an explicit container for declaring multiple instances of `@Sql`.

The following example demonstrates the use of `@Sql` as a repeatable annotation using Java 8. In this scenario the `test-schema.sql` script uses a different syntax for single-line comments.

```
@Test
@Sql(scripts = "/test-schema.sql", config = @SqlConfig(commentPrefix = "`"))
@Sql("/test-user-data.sql")
public void userTest {
	// execute code that uses the test schema and test data
}
```

The following example is identical to the above except that the `@Sql` declarations are grouped together within `@SqlGroup` for compatibility with Java 6 and Java 7.

```
@Test
@SqlGroup({
	@Sql(scripts = "/test-schema.sql", config = @SqlConfig(commentPrefix = "`")),
	@Sql("/test-user-data.sql")
)}
public void userTest {
	// execute code that uses the test schema and test data
}
```

**Script execution phases**

By default, SQL scripts will be executed *before* the corresponding test method. However, if a particular set of scripts needs to be executed *after* the test method — for example, to clean up database state — the `executionPhase` attribute in `@Sql` can be used as seen in the following example. Note that `ISOLATED` and `AFTER_TEST_METHOD` are statically imported from `Sql.TransactionMode` and `Sql.ExecutionPhase` respectively.

```
@Test
@Sql(
	scripts = "create-test-data.sql",
	config = @SqlConfig(transactionMode = ISOLATED)
)
@Sql(
	scripts = "delete-test-data.sql",
	config = @SqlConfig(transactionMode = ISOLATED),
	executionPhase = AFTER_TEST_METHOD
)
public void userTest {
	// execute code that needs the test data to be committed
	// to the database outside of the test's transaction
}
```

**Script configuration with @SqlConfig**

Configuration for script parsing and error handling can be configured via the `@SqlConfig` annotation. When declared as a class-level annotation on an integration test class, `@SqlConfig` serves as *global* configuration for all SQL scripts within the test class hierarchy. When declared directly via the `config` attribute of the `@Sql`annotation, `@SqlConfig` serves as *local* configuration for the SQL scripts declared within the enclosing `@Sql` annotation. Every attribute in `@SqlConfig` has an implicit default value which is documented in the javadocs of the corresponding attribute. Due to the rules defined for annotation attributes in the Java Language Specification, it is unfortunately not possible to assign a value of `null` to an annotation attribute. Thus, in order to support overrides of inherited global configuration, `@SqlConfig`attributes have an explicit default value of either `""` for Strings or `DEFAULT` for Enums. This approach allows local declarations of `@SqlConfig` to selectively override individual attributes from global declarations of `@SqlConfig` by providing a value other than `""` or `DEFAULT`. Global `@SqlConfig` attributes are inherited whenever local `@SqlConfig` attributes do not supply an explicit value other than `""` or `DEFAULT`. Explicit *local* configuration therefore overrides *global* configuration.

The configuration options provided by `@Sql` and `@SqlConfig` are equivalent to those supported by `ScriptUtils` and `ResourceDatabasePopulator` but are a superset of those provided by the `<jdbc:initialize-database/>` XML namespace element. Consult the javadocs of individual attributes in `@Sql` and `@SqlConfig`for details.

**Transaction management for @Sql**

By default, the `SqlScriptsTestExecutionListener` will infer the desired transaction semantics for scripts configured via `@Sql`. Specifically, SQL scripts will be executed without a transaction, within an existing Spring-managed transaction — for example, a transaction managed by the `TransactionalTestExecutionListener`for a test annotated with `@Transactional` — or within an isolated transaction, depending on the configured value of the `transactionMode` attribute in `@SqlConfig`and the presence of a `PlatformTransactionManager` in the test’s `ApplicationContext`. As a bare minimum however, a `javax.sql.DataSource` must be present in the test’s `ApplicationContext`.

If the algorithms used by `SqlScriptsTestExecutionListener` to detect a `DataSource` and `PlatformTransactionManager` and infer the transaction semantics do not suit your needs, you may specify explicit names via the `dataSource` and `transactionManager` attributes of `@SqlConfig`. Furthermore, the transaction propagation behavior can be controlled via the `transactionMode` attribute of `@SqlConfig` — for example, if scripts should be executed in an isolated transaction. Although a thorough discussion of all supported options for transaction management with `@Sql` is beyond the scope of this reference manual, the javadocs for `@SqlConfig` and `SqlScriptsTestExecutionListener` provide detailed information, and the following example demonstrates a typical testing scenario using JUnit 4 and transactional tests with `@Sql`. Note that there is no need to clean up the database after the `usersTest()` method is executed since any changes made to the database (either within the test method or within the `/test-data.sql` script) will be automatically rolled back by the `TransactionalTestExecutionListener` (see[transaction management](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#testcontext-tx) for details).

```
@RunWith(SpringRunner.class)
@ContextConfiguration(classes = TestDatabaseConfig.class)
@Transactional
public class TransactionalSqlScriptsTests {

	protected JdbcTemplate jdbcTemplate;

	@Autowired
	public void setDataSource(DataSource dataSource) {
		this.jdbcTemplate = new JdbcTemplate(dataSource);
	}

	@Test
	@Sql("/test-data.sql")
	public void usersTest() {
		// verify state in test database:
		assertNumUsers(2);
		// execute code that uses the test data...
	}

	protected int countRowsInTable(String tableName) {
		return JdbcTestUtils.countRowsInTable(this.jdbcTemplate, tableName);
	}

	protected void assertNumUsers(int expected) {
		assertEquals("Number of rows in the [user] table.", expected, countRowsInTable("user"));
	}
}
```

### 11.5.9 Parallel test execution

Spring Framework 5.0 introduces basic support for executing tests in parallel within a single JVM when using the *Spring TestContext Framework*. In general this means that most test classes or test methods can be executed in parallel without any changes to test code or configuration.

| ![[Tip]](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/images/tip.png.pagespeed.ce.w22Wv-tZ37.png) |
| ---------------------------------------- |
| For details on how to set up parallel test execution, consult the documentation for your testing framework, build tool, or IDE. |

Keep in mind that the introduction of concurrency into your test suite can result in unexpected side effects, strange runtime behavior, and tests that only fail intermittently or seemingly randomly. The Spring Team therefore provides the following general guidelines for when *not* to execute tests in parallel.

*Do not execute tests in parallel if:*

- Tests make use of Spring’s `@DirtiesContext` support.
- Tests make use of JUnit 4’s `@FixMethodOrder` support or any testing framework feature that is designed to ensure that test methods execute in a particular order. Note, however, that this does not apply if entire test classes are executed in parallel.
- Tests change the state of shared services or systems such as a database, message broker, filesystem, etc. This applies to both in-memory and external systems.

| ![[Tip]](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/images/tip.png.pagespeed.ce.w22Wv-tZ37.png) |
| ---------------------------------------- |
| If parallel test execution fails with an exception stating that the `ApplicationContext` for the current test is no longer active, this typically means that the`ApplicationContext` was removed from the `ContextCache` in a different thread.This may be due to the use of `@DirtiesContext` or due to automatic eviction from the `ContextCache`. If `@DirtiesContext` is the culprit, you will either need to find a way to avoid using `@DirtiesContext` or exclude such tests from parallel execution. If the maximum size of the `ContextCache` has been exceeded, you can increase the maximum size of the cache. See the discussion on [context caching](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#testcontext-ctx-management-caching) for details. |

| ![[Warning]](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/images/warning.png.pagespeed.ce.aG27Rp2bJY.png) |
| ---------------------------------------- |
| Parallel test execution in the Spring TestContext Framework is only possible if the underlying `TestContext` implementation provides a *copy constructor* as explained in the javadocs for `TestContext`. The `DefaultTestContext` used in Spring provides such a constructor; however, if you use a third-party library that provides a custom `TestContext` implementation, you will need to verify if it is suitable for parallel test execution. |

### 11.5.10 TestContext Framework support classes

#### Spring JUnit 4 Runner

The *Spring TestContext Framework* offers full integration with JUnit 4 through a custom runner (supported on JUnit 4.12 or higher). By annotating test classes with`@RunWith(SpringJUnit4ClassRunner.class)` or the shorter `@RunWith(SpringRunner.class)` variant, developers can implement standard JUnit 4 based unit and integration tests and simultaneously reap the benefits of the TestContext framework such as support for loading application contexts, dependency injection of test instances, transactional test method execution, and so on. If you would like to use the Spring TestContext Framework with an alternative runner such as JUnit 4’s `Parameterized` or third-party runners such as the `MockitoJUnitRunner`, you may optionally use [Spring’s support for JUnit rules](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#testcontext-junit4-rules) instead.

The following code listing displays the minimal requirements for configuring a test class to run with the custom Spring `Runner`. `@TestExecutionListeners` is configured with an empty list in order to disable the default listeners, which otherwise would require an `ApplicationContext` to be configured through `@ContextConfiguration`.

```
@RunWith(SpringRunner.class)
@TestExecutionListeners({})
public class SimpleTest {

   @Test
   public void testMethod() {
      // execute test logic...
   }
}
```

#### Spring JUnit 4 Rules

The `org.springframework.test.context.junit4.rules` package provides the following JUnit 4 rules (supported on JUnit 4.12 or higher).

- `SpringClassRule`
- `SpringMethodRule`

`SpringClassRule` is a JUnit `TestRule` that supports *class-level* features of the *Spring TestContext Framework*; whereas, `SpringMethodRule` is a JUnit `MethodRule`that supports instance-level and method-level features of the *Spring TestContext Framework*.

In contrast to the `SpringRunner`, Spring’s rule-based JUnit support has the advantage that it is independent of any `org.junit.runner.Runner` implementation and can therefore be combined with existing alternative runners like JUnit 4’s `Parameterized` or third-party runners such as the `MockitoJUnitRunner`.

In order to support the full functionality of the TestContext framework, a `SpringClassRule` must be combined with a `SpringMethodRule`. The following example demonstrates the proper way to declare these rules in an integration test.

```
// Optionally specify a non-Spring Runner via @RunWith(...)
@ContextConfiguration
public class IntegrationTest {

   @ClassRule
   public static final SpringClassRule SPRING_CLASS_RULE = new SpringClassRule();

   @Rule
   public final SpringMethodRule springMethodRule = new SpringMethodRule();

   @Test
   public void testMethod() {
      // execute test logic...
   }
}
```

#### JUnit 4 support classes

The `org.springframework.test.context.junit4` package provides the following support classes for JUnit 4 based test cases (supported on JUnit 4.12 or higher).

- `AbstractJUnit4SpringContextTests`
- `AbstractTransactionalJUnit4SpringContextTests`

`AbstractJUnit4SpringContextTests` is an abstract base test class that integrates the *Spring TestContext Framework* with explicit `ApplicationContext` testing support in a JUnit 4 environment. When you extend `AbstractJUnit4SpringContextTests`, you can access a `protected` `applicationContext` instance variable that can be used to perform explicit bean lookups or to test the state of the context as a whole.

`AbstractTransactionalJUnit4SpringContextTests` is an abstract *transactional* extension of `AbstractJUnit4SpringContextTests` that adds some convenience functionality for JDBC access. This class expects a `javax.sql.DataSource` bean and a `PlatformTransactionManager` bean to be defined in the `ApplicationContext`. When you extend `AbstractTransactionalJUnit4SpringContextTests` you can access a `protected` `jdbcTemplate` instance variable that can be used to execute SQL statements to query the database. Such queries can be used to confirm database state both *prior to* and *after* execution of database-related application code, and Spring ensures that such queries run in the scope of the same transaction as the application code. When used in conjunction with an ORM tool, be sure to avoid [false positives](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#testcontext-tx-false-positives). As mentioned in [Section 11.3, “JDBC Testing Support”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#integration-testing-support-jdbc), `AbstractTransactionalJUnit4SpringContextTests` also provides convenience methods which delegate to methods in `JdbcTestUtils` using the aforementioned `jdbcTemplate`. Furthermore, `AbstractTransactionalJUnit4SpringContextTests` provides an `executeSqlScript(..)` method for executing SQL scripts against the configured `DataSource`.

| ![[Tip]](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/images/tip.png.pagespeed.ce.w22Wv-tZ37.png) |
| ---------------------------------------- |
| These classes are a convenience for extension. If you do not want your test classes to be tied to a Spring-specific class hierarchy, you can configure your own custom test classes by using `@RunWith(SpringRunner.class)` or [Spring’s JUnit rules](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#testcontext-junit4-rules). |

#### TestNG support classes

The `org.springframework.test.context.testng` package provides the following support classes for TestNG based test cases.

- `AbstractTestNGSpringContextTests`
- `AbstractTransactionalTestNGSpringContextTests`

`AbstractTestNGSpringContextTests` is an abstract base test class that integrates the *Spring TestContext Framework* with explicit `ApplicationContext` testing support in a TestNG environment. When you extend `AbstractTestNGSpringContextTests`, you can access a `protected` `applicationContext` instance variable that can be used to perform explicit bean lookups or to test the state of the context as a whole.

`AbstractTransactionalTestNGSpringContextTests` is an abstract *transactional* extension of `AbstractTestNGSpringContextTests` that adds some convenience functionality for JDBC access. This class expects a `javax.sql.DataSource` bean and a `PlatformTransactionManager` bean to be defined in the `ApplicationContext`. When you extend `AbstractTransactionalTestNGSpringContextTests` you can access a `protected` `jdbcTemplate` instance variable that can be used to execute SQL statements to query the database. Such queries can be used to confirm database state both *prior to* and *after* execution of database-related application code, and Spring ensures that such queries run in the scope of the same transaction as the application code. When used in conjunction with an ORM tool, be sure to avoid [false positives](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#testcontext-tx-false-positives). As mentioned in [Section 11.3, “JDBC Testing Support”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#integration-testing-support-jdbc), `AbstractTransactionalTestNGSpringContextTests` also provides convenience methods which delegate to methods in `JdbcTestUtils` using the aforementioned `jdbcTemplate`. Furthermore, `AbstractTransactionalTestNGSpringContextTests` provides an `executeSqlScript(..)` method for executing SQL scripts against the configured `DataSource`.

| ![[Tip]](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/images/tip.png.pagespeed.ce.w22Wv-tZ37.png) |
| ---------------------------------------- |
| These classes are a convenience for extension. If you do not want your test classes to be tied to a Spring-specific class hierarchy, you can configure your own custom test classes by using `@ContextConfiguration`, `@TestExecutionListeners`, and so on, and by manually instrumenting your test class with a `TestContextManager`. See the source code of `AbstractTestNGSpringContextTests` for an example of how to instrument your test class. |