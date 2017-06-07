## 11.4 **注解**

### 11.4.1 **Spring测试注解**

Spring框架提供以下Spring特定的注解集合，你可以在单元和集成测试中协同TestContext框架使用它们。请参考相应的JAVA帮助文档作进一步了解，包括默认的属性，属性别名等等。

#### @BootstrapWith

`@BootstrapWith`是一个用于配置Spring TestContext框架如何引导的类级别的注解。具体地说，`@BootstrapWith`用于指定一个自定义的`TestContextBootstrapper`。请查看[引导TestContext框架](http://docs.spring.io/spring/docs/5.0.0.BUILD-SNAPSHOT/spring-framework-reference/htmlsingle/#testcontext-bootstrapping)作进一步了解。

#### @ContextConfiguration

`@ContextConfiguration`定义了类级别的元数据来决定如何为集成测试来加载和配置应用程序上下文。具体地说，`@ContextConfiguration`声明了用于加载上下文的应用程序上下文资源路径和注解类。

资源路径通常是类路径中的XML配置文件或者Groovy脚本；而注解类通常是使用`@Configuration`注解的类。但是，资源路径也可以指向文件系统中的文件和脚本，解决类也可能是组件类等等。

```java
@ContextConfiguration("/test-config.xml")
public class XmlApplicationContextTests {
    // class body...
}
```

```java
@ContextConfiguration(classes = TestConfig.class)
public class ConfigClassApplicationContextTests {
    // class body...
}
```

作为声明资源路径或注解类的替代方案或补充，`@ContextConfiguration`可以用于声明`ApplicationContextInitializer`类。

```java
@ContextConfiguration(initializers = CustomContextIntializer.class)
public class ContextInitializerTests {
    // class body...
}
```

`@ContextConfiguration`偶尔也被用作声明`ContextLoader`策略。但注意，通常你不需要显示的配置加载器，因为默认的加载器已经支持资源路径或者注解类以及初始化器。

```java
@ContextConfiguration(locations = "/test-context.xml", loader = CustomContextLoader.class)
public class CustomLoaderXmlApplicationContextTests {
    // class body...
}
```

| ![\[Note\]](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| --- |
| `@ContextConfiguration` 默认对继承父类定义的资源路径或者配置类以及上下文初始化器提供支持。 |

参阅[Section 11.5.4, 上下文管理](http://section%2011.5.xn--4%2C%20context%20management-278prb/)和`@ContextConfiguration`帮助文档作进一步了解。

#### @WebAppConfiguration

`@WebAppConfiguration`是一个用于声明集成测试所加载的`ApplicationContext`须是`WebApplicationContext`的类级别的注解。测试类的`@WebAppConfiguration`注解只是为了保证用于测试的`WebApplicationContext`会被加载，它使用`”file:src/main/webapp”`路径默认值作为web应用的根路径（即，资源基路径）。资源基路径用于幕后创建一个`MockServletContext`作为测试的`WebApplicationContext`的`ServletContext`。

```java
@ContextConfiguration
@WebAppConfiguration
public class WebAppTests {
    // class body...
}
```

要覆盖默认值，请通过隐式值属性指定不同的基本资源路径。 `classpath：`和`file：`资源前缀都被支持。 如果没有提供资源前缀，则假定路径是文件系统资源。

```java
@ContextConfiguration
@WebAppConfiguration("classpath:test-web-resources")
public class WebAppTests {
    // class body...
}
```

注意`@WebAppConfiguration`必须和`@ContextConfiguration`一起使用，或者在同一个测试类，或者在测试类层次结构中。请参阅`@WebAppConfiguration`帮助文档作进一步了解。

#### @ContextHierarchy

`@ContextHierarchy`是一个用于为集成测试定义`ApplicationContext`层次结构的类级别的注解。`@ContextHierarchy`应该声明一个或多个`@ContextConfiguration`实例列表，其中每一个定义上下文层次结构的一个层次。下面的例子展示了在同一个测试类中`@ContextHierarchy`的使用方法。但是，`@ContextHierarchy`一样可以用于测试类的层次结构中。

```java
@ContextHierarchy({
    @ContextConfiguration("/parent-config.xml"),
    @ContextConfiguration("/child-config.xml")
})
public class ContextHierarchyTests {
    // class body...
}
```

```java
@WebAppConfiguration
@ContextHierarchy({
    @ContextConfiguration(classes = AppConfig.class),
    @ContextConfiguration(classes = WebConfig.class)
})
public class WebIntegrationTests {
    // class body...
}
```

如果你想合并或者覆盖一个测试类的层次结构中的应用程序上下文中指定层次的配置，你就必须在类层次中的每一个相应的层次通过为`@ContextConfiguration`的`name`属性提供与该层次相同的值的方式来显示地指定这个层次。请参阅[上下文层次关系](http://docs.spring.io/spring/docs/5.0.0.BUILD-SNAPSHOT/spring-framework-reference/htmlsingle/#testcontext-ctx-management-ctx-hierarchies)和`@ContextHierarchy`帮助文档来获得更多的示例。

#### @ActiveProfiles

`@ActiveProfiles`是一个用于当集成测试加载`ApplicationContext`的时候声明哪一个_bean definition profiles_被激活的类级别的注解。

```java
@ContextConfiguration
@ActiveProfiles("dev")
public class DeveloperTests {
    // class body...
}
```

```java
@ContextConfiguration
@ActiveProfiles({"dev", "integration"})
public class DeveloperIntegrationTests {
    // class body...
}
```

| ![\[Note\]](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| --- |
| `@ActiveProfiles`默认为继承激活的在超类声明的 bean definition profiles提供支持。通过实现一个自定义的[`ActiveProfilesResolver`](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#testcontext-ctx-management-env-profiles-ActiveProfilesResolver)并通过`@ActiveProfiles`的`resolver`属性来注册它的编程的方式来解决激活bean definition profiles问题也是可行的。 |

参阅[使用环境profiles来配置上下文](http://docs.spring.io/spring/docs/5.0.0.BUILD-SNAPSHOT/spring-framework-reference/htmlsingle/#testcontext-ctx-management-env-profiles)和`@ActiveProfiles`帮助文档作进一步了解。

#### @TestPropertySource

`@TestPropertySource`是一个用于为集成测试加载`ApplicationContext`时配置属性文件的位置和增加到`Environment`中的`PropertySources`集中的内联属性的类级别的注解。

测试属性源比那些从系统环境或者Java系统属性以及通过`@PropertySource`或者编程方式声明方式增加的属性源具有更高的优先级。而且，内联属性比从资源路径加载的属性具有更高的优先级。

下面的例子展示了如何从类路径中声明属性文件。

```java
@ContextConfiguration
@TestPropertySource("/test.properties")
public class MyIntegrationTests {
    // class body...
}
```

下面的例子展示了如何声明内联属性。

```java
@ContextConfiguration
@TestPropertySource(properties = { "timezone = GMT", "port: 4242" })
public class MyIntegrationTests {
    // class body...
}
```

#### @DirtiesContext

`@DirtiesContext`指明测试执行期间该Spring `ApplicationContext`已经被弄脏（也就是说通过某种方式被更改或者破坏——比如，更改单例bean的状态）。当应用程序上下文被标为”脏”，它将从测试框架缓存中被移除并关闭。因此，Spring容器将为随后需要同样配置元数据的测试而被重建。

`@DirtiesContext`可以在同一个类或者类层次结构中的类级别和方法级别中使用。在这个场景下，`ApplicationContext`将在任意此注解的方法之前或之后以及当前测试类之前或之后被标为“脏”，这取决于配置的`methodMode`和`classMode`。

下面的例子解释了在多种配置场景下什么时候上下文会被标为“脏”。

* 当在一个类中声明并将类模式设为`BEFORE_CLASS`，则在当前测试类之前。
  ```java
  @DirtiesContext(classMode = BEFORE_CLASS)
  public class FreshContextTests {
      // some tests that require a new Spring container
  }
  ```
* 当在一个类中声明并将类模式设为`AFTER_CLASS`（也就是，默认的类模式），则在当前测试类之后。
  ```java
  @DirtiesContext
  public class ContextDirtyingTests {
      // some tests that result in the Spring container being dirtied
  }
  ```
* 当在一个类中声明并将类模式设为`BEFORE_EACH_TEST_METHOD`，则在当前测试类的每个方法之前。
  ```java
  @DirtiesContext(classMode = BEFORE_EACH_TEST_METHOD)
  public class FreshContextTests {
      // some tests that require a new Spring container
  }
  ```
* 当在一个类中声明并将类模式设为`AFTER_EACH_TEST_METHOD`，则在当前测试类的每个方法之后。
  ```java
  @DirtiesContext(classMode = AFTER_EACH_TEST_METHOD)
  public class ContextDirtyingTests {
      // some tests that result in the Spring container being dirtied
  }
  ```
* 当在一个方法中声明并将方法模式设为`BEFORE_METHOD`，则在当前方法之前。
  ```java
  @DirtiesContext(methodMode = BEFORE_METHOD)
  @Test
  public void testProcessWhichRequiresFreshAppCtx() {
      // some logic that requires a new Spring container
  }
  ```
* 当在一个方法中声明并将方法模式设为`AFTER_METHOD`\(也就是说，默认的方法模式），则在当前方法之后。
  ```java
  @DirtiesContext
  @Test
  public void testProcessWhichDirtiesAppCtx() {
      // some logic that results in the Spring container being dirtied
  }
  ```

如果`@DirtiesContext`被用于上下文被配置为通过`@ContextHierarchy`定义的上下文层次中的一部分的测试中，则`hierarchyMode`标志可用于控制如何声明上下文缓存。默认将使用一个穷举算法用于清除包括不仅当前层次而且与当前测试拥有共同祖先的其它上下文层次的缓存。所有在拥有共同祖先上下文的子层次的应用程序上下文都会从上下文中被移除并关闭。如果穷举算法对于特定的使用场景显得有点威力过猛，那么你可以指定一个更简单的当前层算法来代替，如下所。

```java
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

`@TestExecutionListeners` supports _inherited_ listeners by default. See the javadocs for an example and further details.

#### @Commit

`@Commit` indicates that the transaction for a transactional test method should be _committed_ after the test method has completed. `@Commit` can be used as a direct replacement for `@Rollback(false)` in order to more explicitly convey the intent of the code. Analogous to `@Rollback`, `@Commit` may also be declared as a class-level or method-level annotation.

```
@Commit
@Test
public void testProcessWithoutRollback() {
    // ...
}
```

#### @Rollback

`@Rollback` indicates whether the transaction for a transactional test method should be _rolled back_ after the test method has completed. If `true`, the transaction is rolled back; otherwise, the transaction is committed \(see also `@Commit`\). Rollback semantics for integration tests in the Spring TestContext Framework default to `true` even if`@Rollback` is not explicitly declared.

When declared as a class-level annotation, `@Rollback` defines the default rollback semantics for all test methods within the test class hierarchy. When declared as a method-level annotation, `@Rollback` defines rollback semantics for the specific test method, potentially overriding class-level `@Rollback` or `@Commit` semantics.

```
@Rollback(false)
@Test
public void testProcessWithoutRollback() {
    // ...
}
```

#### @BeforeTransaction

`@BeforeTransaction` indicates that the annotated `void` method should be executed _before_ a transaction is started for test methods configured to run within a transaction via Spring’s `@Transactional` annotation. As of Spring Framework 4.3, `@BeforeTransaction` methods are not required to be `public` and may be declared on Java 8 based interface default methods.

```
@BeforeTransaction
void beforeTransaction() {
    // logic to be executed before a transaction is started
}
```

#### @AfterTransaction

`@AfterTransaction` indicates that the annotated `void` method should be executed _after_ a transaction is ended for test methods configured to run within a transaction via Spring’s `@Transactional` annotation. As of Spring Framework 4.3, `@AfterTransaction` methods are not required to be `public` and may be declared on Java 8 based interface default methods.

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

    @Test
    @Sql(
        scripts = "/test-user-data.sql",
        config = @SqlConfig(commentPrefix = "`", separator = "@@")
    )
    public void userTest {
        // execute code that relies on the test data
    }

#### @SqlGroup

`@SqlGroup` is a container annotation that aggregates several `@Sql` annotations. `@SqlGroup` can be used natively, declaring several nested `@Sql` annotations, or it can be used in conjunction with Java 8’s support for repeatable annotations, where `@Sql` can simply be declared several times on the same class or method, implicitly generating this container annotation.

    @Test
    @SqlGroup({
        @Sql(scripts = "/test-schema.sql", config = @SqlConfig(commentPrefix = "`")),
        @Sql("/test-user-data.sql")
    )}
    public void userTest {
        // execute code that uses the test schema and test data
    }

### 11.4.2 Standard Annotation Support

The following annotations are supported with standard semantics for all configurations of the Spring TestContext Framework. Note that these annotations are not specific to tests and can be used anywhere in the Spring Framework.

* `@Autowired`
* `@Qualifier`
* `@Resource` \(javax.annotation\) _if JSR-250 is present_
* `@ManagedBean` \(javax.annotation\) _if JSR-250 is present_
* `@Inject` \(javax.inject\) _if JSR-330 is present_
* `@Named` \(javax.inject\) _if JSR-330 is present_
* `@PersistenceContext` \(javax.persistence\) _if JPA is present_
* `@PersistenceUnit` \(javax.persistence\) _if JPA is present_
* `@Required`
* `@Transactional`

| ![\[Note\]](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| --- |
| In the Spring TestContext Framework `@PostConstruct` and `@PreDestroy` may be used with standard semantics on any application components configured in the `ApplicationContext`; however, these lifecycle annotations have limited usage within an actual test class.If a method within a test class is annotated with `@PostConstruct`, that method will be executed before any _before_ methods of the underlying test framework \(e.g., methods annotated with JUnit 4’s `@Before`\), and that will apply for every test method in the test class. On the other hand, if a method within a test class is annotated with `@PreDestroy`, that method will _never_ be executed. Within a test class it is therefore recommended to use test lifecycle callbacks from the underlying test framework instead of `@PostConstruct` and `@PreDestroy`. |

### 11.4.3 Spring JUnit 4 Testing Annotations

The following annotations are _only_ supported when used in conjunction with the [SpringRunner](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#testcontext-junit4-runner), [Spring’s JUnit rules](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#testcontext-junit4-rules), or [Spring’s JUnit 4 support classes](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#testcontext-support-classes-junit4).

#### @IfProfileValue

`@IfProfileValue` indicates that the annotated test is enabled for a specific testing environment. If the configured `ProfileValueSource` returns a matching `value` for the provided `name`, the test is enabled. Otherwise, the test will be disabled and effectively _ignored_.

`@IfProfileValue` can be applied at the class level, the method level, or both. Class-level usage of `@IfProfileValue` takes precedence over method-level usage for any methods within that class or its subclasses. Specifically, a test is enabled if it is enabled both at the class level _and_ at the method level; the absence of`@IfProfileValue` means the test is implicitly enabled. This is analogous to the semantics of JUnit 4’s `@Ignore` annotation, except that the presence of `@Ignore`always disables a test.

```
@IfProfileValue(name="java.vendor", value="Oracle Corporation")
@Test
public void testProcessWhichRunsOnlyOnOracleJvm() {
    // some logic that should run only on Java VMs from Oracle Corporation
}
```

Alternatively, you can configure `@IfProfileValue` with a list of `values` \(with _OR_ semantics\) to achieve TestNG-like support for _test groups_ in a JUnit 4 environment. Consider the following example:

```
@IfProfileValue(name="test-groups", values={"unit-tests", "integration-tests"})
@Test
public void testProcessWhichRunsForUnitOrIntegrationTestGroups() {
    // some logic that should run only for unit and integration test groups
}
```

#### @ProfileValueSourceConfiguration

`@ProfileValueSourceConfiguration` is a class-level annotation that specifies what type of `ProfileValueSource` to use when retrieving _profile values_ configured through the `@IfProfileValue` annotation. If `@ProfileValueSourceConfiguration` is not declared for a test, `SystemProfileValueSource` is used by default.

```
@ProfileValueSourceConfiguration(CustomProfileValueSource.class)
public class CustomProfileValueSourceTests {
    // class body...
}
```

#### @Timed

`@Timed` indicates that the annotated test method must finish execution in a specified time period \(in milliseconds\). If the text execution time exceeds the specified time period, the test fails.

The time period includes execution of the test method itself, any repetitions of the test \(see `@Repeat`\), as well as any _set up_ or _tear down_ of the test fixture.

```
@Timed(millis=1000)
public void testProcessWithOneSecondTimeout() {
    // some logic that should not take longer than 1 second to execute
}
```

Spring’s `@Timed` annotation has different semantics than JUnit 4’s `@Test(timeout=…)` support. Specifically, due to the manner in which JUnit 4 handles test execution timeouts \(that is, by executing the test method in a separate `Thread`\), `@Test(timeout=…)` preemptively fails the test if the test takes too long. Spring’s `@Timed`, on the other hand, does not preemptively fail the test but rather waits for the test to complete before failing.

#### @Repeat

`@Repeat` indicates that the annotated test method must be executed repeatedly. The number of times that the test method is to be executed is specified in the annotation.

The scope of execution to be repeated includes execution of the test method itself as well as any _set up_ or _tear down_ of the test fixture.

```
@Repeat(10)
@Test
public void testProcessRepeatedly() {
    // ...
}
```

### 11.4.4 Meta-Annotation Support for Testing

It is possible to use most test-related annotations as [meta-annotations](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#beans-meta-annotations) in order to create custom _composed annotations_ and reduce configuration duplication across a test suite.

Each of the following may be used as meta-annotations in conjunction with the [TestContext framework](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#testcontext-framework).

* `@BootstrapWith`
* `@ContextConfiguration`
* `@ContextHierarchy`
* `@ActiveProfiles`
* `@TestPropertySource`
* `@DirtiesContext`
* `@WebAppConfiguration`
* `@TestExecutionListeners`
* `@Transactional`
* `@BeforeTransaction`
* `@AfterTransaction`
* `@Commit`
* `@Rollback`
* `@Sql`
* `@SqlConfig`
* `@SqlGroup`
* `@Repeat`
* `@Timed`
* `@IfProfileValue`
* `@ProfileValueSourceConfiguration`

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

We can reduce the above duplication by introducing a custom _composed annotation_ that centralizes the common test configuration like this:

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

