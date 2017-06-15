## 11.4 **注解**

### 11.4.1 **Spring测试注解**

Spring框架提供以下Spring特定的注解集合，你可以在单元和集成测试中协同TestContext框架使用它们。请参考相应的JAVA帮助文档作进一步了解，包括默认的属性，属性别名等等。

#### @BootstrapWith

`@BootstrapWith`是一个用于配置Spring TestContext框架如何引导的类级别的注解。具体地说，`@BootstrapWith`用于指定一个自定义的`TestContextBootstrapper`。请查看[引导TestContext框架](http://docs.spring.io/spring/docs/5.0.0.BUILD-SNAPSHOT/spring-framework-reference/htmlsingle/#testcontext-bootstrapping)作进一步了解。

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

参阅[Section 11.5.4, 上下文管理](http://section 11.5.xn--4%2C context management-278prb/)和`@ContextConfiguration`帮助文档作进一步了解。

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

参阅`DirtiesContext.HierarchyMode`帮助文档以获得`EXHAUSTIVE`和`CURRENT_LEVEL`算法更详细的了解。

#### @TestExecutionListeners

`@TestExecutionListeners`定义了一个类级别的元数据，用于配置需要用`TestContextManager`进行注册的`TestExecutionListener`实现。通常，`@TestExecutionListeners`与`@ContextConfiguration`一起使用。

```java
@ContextConfiguration
@TestExecutionListeners({CustomTestExecutionListener.class, AnotherTestExecutionListener.class})
public class CustomTestExecutionListenerTests {
    // class body...
}
```

`@TestExecutionListeners`默认支持继承监听器。参阅帮助文档获得示例和更详细的了解。

#### @Commit

`@Commit`指定事务性的测试方法在测试方法执行完成后对事务进行提交。`@Commit`可以用作`@Rollback(false)`的直接替代，以更好的传达代码的意图。和`@Rollback`一样，`@Commit`可以在类层次或者方法层级声明。

```java
@Commit
@Test
public void testProcessWithoutRollback() {
    // ...
}
```

#### @Rollback

`@Rollback`指明当测试方法执行完毕的时候是否对事务性方法中的事务进行回滚。如果为`true`,则进行回滚；否则，则提交（请参加`@Commit`）。在Spring TestContext框架中，集成测试默认的`@Rollback`语义为`true`，即使你不显示的指定它。

当声明为类级别注解时，`@Rollback`为测试类层次结构中的所有测试方法定义了默认回滚语义。 当被声明为方法级别的注解，则`@Rollback`为特定的方法指定回滚语义，并覆盖类级别的`@Rollback`和`@Commit`语义。

```java
@Rollback(false)
@Test
public void testProcessWithoutRollback() {
    // ...
}
```

#### @BeforeTransaction

`@BeforeTransaction`指明通过Spring的`@Transactional`注解配置为需要在事务中执行的测试方法在事务开始之前先执行注解的`void`方法。从Spring框架4.3版本起，`@BeforeTransaction`方法不再需要为`public`并可能被声明为基于Java8的接口的默认方法。

```java
@BeforeTransaction
void beforeTransaction() {
    // logic to be executed before a transaction is started
}
```

#### @AfterTransaction

`@AfterTransaction`指明通过Spring的`@Transactional`注解配置为需要在事务中执行的测试方法在事务结束之后执行注解的`void`方法。从Spring框架4.3版本起，`@AfterTransaction`方法不再需要为`public`并可能被声明为基于Java8的接口的默认方法。

```java
@AfterTransaction
void afterTransaction() {
    // logic to be executed after a transaction has ended
}
```

#### @Sql

`@Sql`用于注解测试类或者测试方法，以让在集成测试过程中配置的SQL脚本能够在给定的的数据库中得到执行。

```java
@Test
@Sql({"/test-schema.sql", "/test-user-data.sql"})
public void userTest {
    // execute code that relies on the test schema and test data
}
```

请参阅[通过@sql声明执行的SQL脚本](http://docs.spring.io/spring/docs/5.0.0.BUILD-SNAPSHOT/spring-framework-reference/htmlsingle/#testcontext-executing-sql-declaratively)作进一步了解。

#### @SqlConfig

`@SqlConfig`定义了用于决定如何解析和执行通过`@Sql`注解配置的SQL脚本。

```java
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

`@SqlGroup`是一个用于聚合几个`@Sql`注解的容器注解。`@SqlGroup`可以直接使用，通过声明几个嵌套的`@Sql`注解，也可以与Java8的可重复注解支持协同使用，即简单地在同一个类或方法上声明几个`@Sql`注解，隐式地产生这个容器注解。

    @Test
    @SqlGroup({
        @Sql(scripts = "/test-schema.sql", config = @SqlConfig(commentPrefix = "`")),
        @Sql("/test-user-data.sql")
    )}
    public void userTest {
        // execute code that uses the test schema and test data
    }

### 11.4.2 标准注解支持

以下注解为Spring TestContext 框架所有的配置提供标准语义支持。注意这些注解不仅限于测试，可以用在Spring框架的任意地方。

* `@Autowired`
* `@Qualifier`
* `@Resource` \(javax.annotation\)如果_JSR-250_存在
* `@ManagedBean` \(javax.annotation\)如果_JSR-250_存在
* `@Inject` \(javax.inject\) 如果_JSR-330_存在
* `@Named` \(javax.inject\)如果_JSR-330_存在
* `@PersistenceContext` \(javax.persistence\) 如果JPA存在
* `@PersistenceUnit` \(javax.persistence\) 如果JPA存在
* `@Required`
* `@Transactional`

| ![\[Note\]](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| --- |
| 在Spring TestContext 框架中，`@PostConstruct`和 `@PreDestroy` 可以通过标准语义在配置于`ApplicationContext`的任意应用程序组件中使用; 但是, 这些生命周期注解在实际测试类中只有很有限的作用。如果一个测试类的方法被注解为`@PostConstruct`，这个方法将在test框架中的任何before方法（也就是被JUnit中的`@Before`注解方法）调用之前被执行, 这个规则将被应用于测试类的每个方法。另一方面，如果一个测试类的方法被注解为 `@PreDestroy`，这个方法将永远不会被执行。因为建议在测试类中使用test 框架的测试生命周期回调来代替使用`@PostConstruct` 和 `@PreDestroy`。 |

### 11.4.3 Spring JUnit 4 测试注解

仅当与[SpringRunner](#)，[Spring’s JUnit rules](#)或[Spring’s JUnit 4 support classes](#)结合使用时，才支持以下注释。

#### @IfProfileValue

`@IfProfileValue`指明该测试只在特定的测试环境中被启用。如果配置的`ProfileValueSource`为所提供的`name`返回匹配的`value`，这该测试将被启用。否则，该测试将被禁用并忽略。

`@IfProfileValue`可以用在类级别、方法级别或者两个同时。使用类级别的`@IfProfileValue`注解优先于当前类或其子类的任意方法的使用方法级别的注解。有`@IfProfileValue`注解意味着则测试被隐式开启。这与JUnit4的`@Ignore`注解是相类似的，除了使用`@Ignore`注解是用于禁用测试的之外。

```java
@IfProfileValue(name="java.vendor", value="Oracle Corporation")
@Test
public void testProcessWhichRunsOnlyOnOracleJvm() {
    // some logic that should run only on Java VMs from Oracle Corporation
}
```

或者，你可以 配置`@IfProfileValue`使用`values`列表（或语义）来实现JUnit 4环境中的类似TestNG对测试组的支持。

```java
@IfProfileValue(name="test-groups", values={"unit-tests", "integration-tests"})
@Test
public void testProcessWhichRunsForUnitOrIntegrationTestGroups() {
    // some logic that should run only for unit and integration test groups
}
```

#### @ProfileValueSourceConfiguration

`@ProfileValueSourceConfiguration`是类级别注解，用于当获取通过`@IfProfileValue`配置的profile值时指定使用什么样的`ProfileValueSource`类型。如果一个测试没有指定`@ProfileValueSourceConfiguration`，那么默认使用`SystemProfileValueSource`。

```java
@ProfileValueSourceConfiguration(CustomProfileValueSource.class)
public class CustomProfileValueSourceTests {
    // class body...
}
```

#### @Timed

`@Timed`用于指明被注解的测试必须在指定的时限（毫秒）内结束。如果测试超过指定时限，就当作测试失败。

时限包括测试方法本身所耗费的时间，包括任何重复（请查看`@Repeat`）及任意初始化和销毁所用的时间。

```java
@Timed(millis=1000)
public void testProcessWithOneSecondTimeout() {
    // some logic that should not take longer than 1 second to execute
}
```

Spring的`@Timed`注解与JUnit 4的`@Test(timeout=…​)`支持相比具有不同的语义。确切地说，由于在JUnit 4中处理方法执行超时的方式（也就是，在独立纯程中执行该测试方法），如果一个测试方法执行时间太长，`@Test(timeout=…​)`将直接判定该测试失败。而Spring的`@Timed`则不直接判定失败而是等待测试完成。

#### @Repeat

`@Repeat`指明该测试方法需被重复执行。注解指定该测试方法被重复的次数。

重复的范围包括该测试方法自身也包括相应的初始化和销毁方法。

```java
@Repeat(10)
@Test
public void testProcessRepeatedly() {
    // ...
}
```

### 11.4.4 Meta-Annotation Support for Testing

可以将大部分测试相关的注解当作[meta-annotations](http://docs.spring.io/spring/docs/5.0.0.BUILD-SNAPSHOT/spring-framework-reference/htmlsingle/#beans-meta-annotations)使用，以创建自定义组合注解来减少测试集中的重复配置。

下面的每个都可以在[TestContext](http://docs.spring.io/spring/docs/5.0.0.BUILD-SNAPSHOT/spring-framework-reference/htmlsingle/#testcontext-framework)框架中被当作meta-annotations使用。

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

例如，如果发现我们在基于JUnit 4的测试集中重复以下配置…

```java
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

我们可以通过一个自定义的组合注解来减少上述的重复量，将通用的测试配置集中起来，就像这样：

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@ContextConfiguration({"/app-config.xml", "/test-data-access-config.xml"})
@ActiveProfiles("dev")
@Transactional
public @interface TransactionalDevTest { }
```

然后我们就可以像下面一样使用我们自定义的`@TransactionalDevTest`注解来简化每个类的配置：

```java
@RunWith(SpringRunner.class)
@TransactionalDevTest
public class OrderRepositoryTests { }

@RunWith(SpringRunner.class)
@TransactionalDevTest
public class UserRepositoryTests { }
```

想获得详情，请查看[Spring注解编程模型](http://docs.spring.io/spring/docs/5.0.0.BUILD-SNAPSHOT/spring-framework-reference/htmlsingle/#annotation-programming-model)

