## 11.4 Annotations

### 11.4.1 Spring Testing Annotations

The Spring Framework provides the following set of *Spring-specific* annotations that you can use in your unit and integration tests in conjunction with the TestContext framework. Refer to the corresponding javadocs for further information, including default attribute values, attribute aliases, and so on.

#### @BootstrapWith

`@BootstrapWith` is a class-level annotation that is used to configure how the *Spring TestContext Framework* is bootstrapped. Specifically, `@BootstrapWith` is used to specify a custom `TestContextBootstrapper`. Consult the [Bootstrapping the TestContext framework](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#testcontext-bootstrapping) section for further details.

#### @ContextConfiguration

`@ContextConfiguration` defines class-level metadata that is used to determine how to load and configure an `ApplicationContext` for integration tests. Specifically,`@ContextConfiguration` declares the application context resource `locations` or the annotated `classes` that will be used to load the context.

Resource locations are typically XML configuration files or Groovy scripts located in the classpath; whereas, annotated classes are typically `@Configuration` classes. However, resource locations can also refer to files and scripts in the file system, and annotated classes can be component classes, etc.

```
@ContextConfiguration("/test-config.xml")
public class XmlApplicationContextTests {
	// class body...
}
```

```
@ContextConfiguration(classes = TestConfig.class)
public class ConfigClassApplicationContextTests {
	// class body...
}
```

As an alternative or in addition to declaring resource locations or annotated classes, `@ContextConfiguration` may be used to declare `ApplicationContextInitializer` classes.

```
@ContextConfiguration(initializers = CustomContextIntializer.class)
public class ContextInitializerTests {
	// class body...
}
```

`@ContextConfiguration` may optionally be used to declare the `ContextLoader` strategy as well. Note, however, that you typically do not need to explicitly configure the loader since the default loader supports either resource `locations` or annotated `classes` as well as `initializers`.

```
@ContextConfiguration(locations = "/test-context.xml", loader = CustomContextLoader.class)
public class CustomLoaderXmlApplicationContextTests {
	// class body...
}
```

| ![[Note]](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| ---------------------------------------- |
| `@ContextConfiguration` provides support for *inheriting* resource locations or configuration classes as well as context initializers declared by superclasses by default. |

See [Section 11.5.4, “Context management”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#testcontext-ctx-management) and the `@ContextConfiguration` javadocs for further details.

#### @WebAppConfiguration

`@WebAppConfiguration` is a class-level annotation that is used to declare that the `ApplicationContext` loaded for an integration test should be a `WebApplicationContext`. The mere presence of `@WebAppConfiguration` on a test class ensures that a `WebApplicationContext` will be loaded for the test, using the default value of `"file:src/main/webapp"` for the path to the root of the web application (i.e., the *resource base path*). The resource base path is used behind the scenes to create a `MockServletContext` which serves as the `ServletContext` for the test’s `WebApplicationContext`.

```
@ContextConfiguration
@WebAppConfiguration
public class WebAppTests {
	// class body...
}
```

To override the default, specify a different base resource path via the *implicit* `value` attribute. Both `classpath:` and `file:` resource prefixes are supported. If no resource prefix is supplied the path is assumed to be a file system resource.

```
@ContextConfiguration
@WebAppConfiguration("classpath:test-web-resources")
public class WebAppTests {
	// class body...
}
```

Note that `@WebAppConfiguration` must be used in conjunction with `@ContextConfiguration`, either within a single test class or within a test class hierarchy. See the `@WebAppConfiguration` javadocs for further details.

#### @ContextHierarchy

`@ContextHierarchy` is a class-level annotation that is used to define a hierarchy of `ApplicationContext`s for integration tests. `@ContextHierarchy` should be declared with a list of one or more `@ContextConfiguration` instances, each of which defines a level in the context hierarchy. The following examples demonstrate the use of `@ContextHierarchy` within a single test class; however, `@ContextHierarchy` can also be used within a test class hierarchy.

```
@ContextHierarchy({
	@ContextConfiguration("/parent-config.xml"),
	@ContextConfiguration("/child-config.xml")
})
public class ContextHierarchyTests {
	// class body...
}
```

```
@WebAppConfiguration
@ContextHierarchy({
	@ContextConfiguration(classes = AppConfig.class),
	@ContextConfiguration(classes = WebConfig.class)
})
public class WebIntegrationTests {
	// class body...
}
```

If you need to merge or override the configuration for a given level of the context hierarchy within a test class hierarchy, you must explicitly name that level by supplying the same value to the `name` attribute in `@ContextConfiguration` at each corresponding level in the class hierarchy. See [the section called “Context hierarchies”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#testcontext-ctx-management-ctx-hierarchies) and the `@ContextHierarchy` javadocs for further examples.

#### @ActiveProfiles

`@ActiveProfiles` is a class-level annotation that is used to declare which *bean definition profiles* should be active when loading an `ApplicationContext` for an integration test.

```
@ContextConfiguration
@ActiveProfiles("dev")
public class DeveloperTests {
	// class body...
}
```

```
@ContextConfiguration
@ActiveProfiles({"dev", "integration"})
public class DeveloperIntegrationTests {
	// class body...
}
```

| ![[Note]](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| ---------------------------------------- |
| `@ActiveProfiles` provides support for *inheriting* active bean definition profiles declared by superclasses by default. It is also possible to resolve active bean definition profiles programmatically by implementing a custom [`ActiveProfilesResolver`](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#testcontext-ctx-management-env-profiles-ActiveProfilesResolver) and registering it via the `resolver` attribute of `@ActiveProfiles`. |

See [the section called “Context configuration with environment profiles”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#testcontext-ctx-management-env-profiles) and the `@ActiveProfiles` javadocs for examples and further details.

#### @TestPropertySource

`@TestPropertySource` is a class-level annotation that is used to configure the locations of properties files and inlined properties to be added to the set of `PropertySources` in the `Environment` for an `ApplicationContext` loaded for an integration test.

Test property sources have higher precedence than those loaded from the operating system’s environment or Java system properties as well as property sources added by the application declaratively via `@PropertySource` or programmatically. Thus, test property sources can be used to selectively override properties defined in system and application property sources. Furthermore, inlined properties have higher precedence than properties loaded from resource locations.

The following example demonstrates how to declare a properties file from the classpath.

```
@ContextConfiguration
@TestPropertySource("/test.properties")
public class MyIntegrationTests {
	// class body...
}
```

The following example demonstrates how to declare *inlined* properties.

```
@ContextConfiguration
@TestPropertySource(properties = { "timezone = GMT", "port: 4242" })
public class MyIntegrationTests {
	// class body...
}
```

#### @DirtiesContext

`@DirtiesContext` indicates that the underlying Spring `ApplicationContext` has been *dirtied* during the execution of a test (i.e., modified or corrupted in some manner — for example, by changing the state of a singleton bean) and should be closed. When an application context is marked *dirty*, it is removed from the testing framework’s cache and closed. As a consequence, the underlying Spring container will be rebuilt for any subsequent test that requires a context with the same configuration metadata.

`@DirtiesContext` can be used as both a class-level and method-level annotation within the same class or class hierarchy. In such scenarios, the `ApplicationContext` is marked as *dirty* before or after any such annotated method as well as before or after the current test class, depending on the configured `methodMode` and `classMode`.

The following examples explain when the context would be dirtied for various configuration scenarios:

- Before the current test class, when declared on a class with class mode set to `BEFORE_CLASS`.

  ```
  @DirtiesContext(classMode = BEFORE_CLASS)
  public class FreshContextTests {
  	// some tests that require a new Spring container
  }
  ```

- After the current test class, when declared on a class with class mode set to `AFTER_CLASS` (i.e., the default class mode).

  ```
  @DirtiesContext
  public class ContextDirtyingTests {
  	// some tests that result in the Spring container being dirtied
  }
  ```

- Before each test method in the current test class, when declared on a class with class mode set to `BEFORE_EACH_TEST_METHOD.`

  ```
  @DirtiesContext(classMode = BEFORE_EACH_TEST_METHOD)
  public class FreshContextTests {
  	// some tests that require a new Spring container
  }
  ```

- After each test method in the current test class, when declared on a class with class mode set to `AFTER_EACH_TEST_METHOD.`

  ```
  @DirtiesContext(classMode = AFTER_EACH_TEST_METHOD)
  public class ContextDirtyingTests {
  	// some tests that result in the Spring container being dirtied
  }
  ```

- Before the current test, when declared on a method with the method mode set to `BEFORE_METHOD`.

  ```
  @DirtiesContext(methodMode = BEFORE_METHOD)
  @Test
  public void testProcessWhichRequiresFreshAppCtx() {
  	// some logic that requires a new Spring container
  }
  ```

- After the current test, when declared on a method with the method mode set to `AFTER_METHOD` (i.e., the default method mode).

  ```
  @DirtiesContext
  @Test
  public void testProcessWhichDirtiesAppCtx() {
  	// some logic that results in the Spring container being dirtied
  }
  ```

If `@DirtiesContext` is used in a test whose context is configured as part of a context hierarchy via `@ContextHierarchy`, the `hierarchyMode` flag can be used to control how the context cache is cleared. By default an *exhaustive* algorithm will be used that clears the context cache including not only the current level but also all other context hierarchies that share an ancestor context common to the current test; all `ApplicationContext`s that reside in a sub-hierarchy of the common ancestor context will be removed from the context cache and closed. If the *exhaustive* algorithm is overkill for a particular use case, the simpler *current level* algorithm can be specified instead, as seen below.

```
@ContextHierarchy({
	@ContextConfiguration("/parent-config.xml"),
	@ContextConfiguration("/child-config.xml")
})
public class BaseTests {
	// class body...
}

public class ExtendedTests extends BaseTests {

	@Test
	@DirtiesContext(hierarchyMode = CURRENT_LEVEL)
	public void test() {
		// some logic that results in the child context being dirtied
	}
}
```

For further details regarding the `EXHAUSTIVE` and `CURRENT_LEVEL` algorithms see the `DirtiesContext.HierarchyMode` javadocs.

#### @TestExecutionListeners

`@TestExecutionListeners` defines class-level metadata for configuring the `TestExecutionListener` implementations that should be registered with the`TestContextManager`. Typically, `@TestExecutionListeners` is used in conjunction with `@ContextConfiguration`.

```
@ContextConfiguration
@TestExecutionListeners({CustomTestExecutionListener.class, AnotherTestExecutionListener.class})
public class CustomTestExecutionListenerTests {
	// class body...
}
```

`@TestExecutionListeners` supports *inherited* listeners by default. See the javadocs for an example and further details.

#### @Commit

`@Commit` indicates that the transaction for a transactional test method should be *committed* after the test method has completed. `@Commit` can be used as a direct replacement for `@Rollback(false)` in order to more explicitly convey the intent of the code. Analogous to `@Rollback`, `@Commit` may also be declared as a class-level or method-level annotation.

```
@Commit
@Test
public void testProcessWithoutRollback() {
	// ...
}
```

#### @Rollback

`@Rollback` indicates whether the transaction for a transactional test method should be *rolled back* after the test method has completed. If `true`, the transaction is rolled back; otherwise, the transaction is committed (see also `@Commit`). Rollback semantics for integration tests in the Spring TestContext Framework default to `true` even if`@Rollback` is not explicitly declared.

When declared as a class-level annotation, `@Rollback` defines the default rollback semantics for all test methods within the test class hierarchy. When declared as a method-level annotation, `@Rollback` defines rollback semantics for the specific test method, potentially overriding class-level `@Rollback` or `@Commit` semantics.

```
@Rollback(false)
@Test
public void testProcessWithoutRollback() {
	// ...
}
```

#### @BeforeTransaction

`@BeforeTransaction` indicates that the annotated `void` method should be executed *before* a transaction is started for test methods configured to run within a transaction via Spring’s `@Transactional` annotation. As of Spring Framework 4.3, `@BeforeTransaction` methods are not required to be `public` and may be declared on Java 8 based interface default methods.

```
@BeforeTransaction
void beforeTransaction() {
	// logic to be executed before a transaction is started
}
```

#### @AfterTransaction

`@AfterTransaction` indicates that the annotated `void` method should be executed *after* a transaction is ended for test methods configured to run within a transaction via Spring’s `@Transactional` annotation. As of Spring Framework 4.3, `@AfterTransaction` methods are not required to be `public` and may be declared on Java 8 based interface default methods.

```
@AfterTransaction
void afterTransaction() {
	// logic to be executed after a transaction has ended
}
```

#### @Sql

`@Sql` is used to annotate a test class or test method to configure SQL scripts to be executed against a given database during integration tests.

```
@Test
@Sql({"/test-schema.sql", "/test-user-data.sql"})
public void userTest {
	// execute code that relies on the test schema and test data
}
```

See [the section called “Executing SQL scripts declaratively with @Sql”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#testcontext-executing-sql-declaratively) for further details.

#### @SqlConfig

`@SqlConfig` defines metadata that is used to determine how to parse and execute SQL scripts configured via the `@Sql` annotation.

```
@Test
@Sql(
	scripts = "/test-user-data.sql",
	config = @SqlConfig(commentPrefix = "`", separator = "@@")
)
public void userTest {
	// execute code that relies on the test data
}
```

#### @SqlGroup

`@SqlGroup` is a container annotation that aggregates several `@Sql` annotations. `@SqlGroup` can be used natively, declaring several nested `@Sql` annotations, or it can be used in conjunction with Java 8’s support for repeatable annotations, where `@Sql` can simply be declared several times on the same class or method, implicitly generating this container annotation.

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

### 11.4.2 Standard Annotation Support

The following annotations are supported with standard semantics for all configurations of the Spring TestContext Framework. Note that these annotations are not specific to tests and can be used anywhere in the Spring Framework.

- `@Autowired`
- `@Qualifier`
- `@Resource` (javax.annotation) *if JSR-250 is present*
- `@ManagedBean` (javax.annotation) *if JSR-250 is present*
- `@Inject` (javax.inject) *if JSR-330 is present*
- `@Named` (javax.inject) *if JSR-330 is present*
- `@PersistenceContext` (javax.persistence) *if JPA is present*
- `@PersistenceUnit` (javax.persistence) *if JPA is present*
- `@Required`
- `@Transactional`

| ![[Note]](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| ---------------------------------------- |
| In the Spring TestContext Framework `@PostConstruct` and `@PreDestroy` may be used with standard semantics on any application components configured in the `ApplicationContext`; however, these lifecycle annotations have limited usage within an actual test class.If a method within a test class is annotated with `@PostConstruct`, that method will be executed before any *before* methods of the underlying test framework (e.g., methods annotated with JUnit 4’s `@Before`), and that will apply for every test method in the test class. On the other hand, if a method within a test class is annotated with `@PreDestroy`, that method will *never* be executed. Within a test class it is therefore recommended to use test lifecycle callbacks from the underlying test framework instead of `@PostConstruct` and `@PreDestroy`. |

### 11.4.3 Spring JUnit 4 Testing Annotations

The following annotations are *only* supported when used in conjunction with the [SpringRunner](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#testcontext-junit4-runner), [Spring’s JUnit rules](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#testcontext-junit4-rules), or [Spring’s JUnit 4 support classes](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#testcontext-support-classes-junit4).

#### @IfProfileValue

`@IfProfileValue` indicates that the annotated test is enabled for a specific testing environment. If the configured `ProfileValueSource` returns a matching `value` for the provided `name`, the test is enabled. Otherwise, the test will be disabled and effectively *ignored*.

`@IfProfileValue` can be applied at the class level, the method level, or both. Class-level usage of `@IfProfileValue` takes precedence over method-level usage for any methods within that class or its subclasses. Specifically, a test is enabled if it is enabled both at the class level *and* at the method level; the absence of`@IfProfileValue` means the test is implicitly enabled. This is analogous to the semantics of JUnit 4’s `@Ignore` annotation, except that the presence of `@Ignore`always disables a test.

```
@IfProfileValue(name="java.vendor", value="Oracle Corporation")
@Test
public void testProcessWhichRunsOnlyOnOracleJvm() {
	// some logic that should run only on Java VMs from Oracle Corporation
}
```

Alternatively, you can configure `@IfProfileValue` with a list of `values` (with *OR* semantics) to achieve TestNG-like support for *test groups* in a JUnit 4 environment. Consider the following example:

```
@IfProfileValue(name="test-groups", values={"unit-tests", "integration-tests"})
@Test
public void testProcessWhichRunsForUnitOrIntegrationTestGroups() {
	// some logic that should run only for unit and integration test groups
}
```

#### @ProfileValueSourceConfiguration

`@ProfileValueSourceConfiguration` is a class-level annotation that specifies what type of `ProfileValueSource` to use when retrieving *profile values* configured through the `@IfProfileValue` annotation. If `@ProfileValueSourceConfiguration` is not declared for a test, `SystemProfileValueSource` is used by default.

```
@ProfileValueSourceConfiguration(CustomProfileValueSource.class)
public class CustomProfileValueSourceTests {
	// class body...
}
```

#### @Timed

`@Timed` indicates that the annotated test method must finish execution in a specified time period (in milliseconds). If the text execution time exceeds the specified time period, the test fails.

The time period includes execution of the test method itself, any repetitions of the test (see `@Repeat`), as well as any *set up* or *tear down* of the test fixture.

```
@Timed(millis=1000)
public void testProcessWithOneSecondTimeout() {
	// some logic that should not take longer than 1 second to execute
}
```

Spring’s `@Timed` annotation has different semantics than JUnit 4’s `@Test(timeout=…)` support. Specifically, due to the manner in which JUnit 4 handles test execution timeouts (that is, by executing the test method in a separate `Thread`), `@Test(timeout=…)` preemptively fails the test if the test takes too long. Spring’s `@Timed`, on the other hand, does not preemptively fail the test but rather waits for the test to complete before failing.

#### @Repeat

`@Repeat` indicates that the annotated test method must be executed repeatedly. The number of times that the test method is to be executed is specified in the annotation.

The scope of execution to be repeated includes execution of the test method itself as well as any *set up* or *tear down* of the test fixture.

```
@Repeat(10)
@Test
public void testProcessRepeatedly() {
	// ...
}
```

### 11.4.4 Meta-Annotation Support for Testing

It is possible to use most test-related annotations as [meta-annotations](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#beans-meta-annotations) in order to create custom *composed annotations* and reduce configuration duplication across a test suite.

Each of the following may be used as meta-annotations in conjunction with the [TestContext framework](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#testcontext-framework).

- `@BootstrapWith`
- `@ContextConfiguration`
- `@ContextHierarchy`
- `@ActiveProfiles`
- `@TestPropertySource`
- `@DirtiesContext`
- `@WebAppConfiguration`
- `@TestExecutionListeners`
- `@Transactional`
- `@BeforeTransaction`
- `@AfterTransaction`
- `@Commit`
- `@Rollback`
- `@Sql`
- `@SqlConfig`
- `@SqlGroup`
- `@Repeat`
- `@Timed`
- `@IfProfileValue`
- `@ProfileValueSourceConfiguration`

For example, if we discover that we are repeating the following configuration across our JUnit 4 based test suite…

```
@RunWith(SpringRunner.class)
@ContextConfiguration({"/app-config.xml", "/test-data-access-config.xml"})
@ActiveProfiles("dev")
@Transactional
public class OrderRepositoryTests { }

@RunWith(SpringRunner.class)
@ContextConfiguration({"/app-config.xml", "/test-data-access-config.xml"})
@ActiveProfiles("dev")
@Transactional
public class UserRepositoryTests { }
```

We can reduce the above duplication by introducing a custom *composed annotation* that centralizes the common test configuration like this:

```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@ContextConfiguration({"/app-config.xml", "/test-data-access-config.xml"})
@ActiveProfiles("dev")
@Transactional
public @interface TransactionalDevTest { }
```

Then we can use our custom `@TransactionalDevTest` annotation to simplify the configuration of individual test classes as follows:

```
@RunWith(SpringRunner.class)
@TransactionalDevTest
public class OrderRepositoryTests { }

@RunWith(SpringRunner.class)
@TransactionalDevTest
public class UserRepositoryTests { }
```

For further details, consult the [Spring Annotation Programming Model](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#annotation-programming-model).