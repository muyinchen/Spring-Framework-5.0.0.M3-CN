## 11.2 **集成测试的目标**

Spring的集成测试支持有以下几点主要目标：

* 管理各个测试执行之间的[Spring IoC容器缓存](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/integration-testing.html#testing-ctx-management)

* 提供[测试配置实例的依赖注入](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/integration-testing.html#testing-fixture-di)

* 提供适合集成测试的[事务管理](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/integration-testing.html#testing-tx)

* 提供辅助开发人员编写集成测试的[具备Spring特性的基础类](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/integration-testing.html#testing-support-classes)

下面几节将解释每个目标并提供实现和配置详情的链接。

### 11.2.1 **上下文管理和缓存**

Spring TestContex框架对Spring `ApplicationContext `和`WebApplicationContext`提供一致性加载并对它们进行缓存。对加载的上下文进行缓存提供支持是很重要的，因为启动时间是个问题——不是因为Spring自己的开销，而是被Spring容器初始化的对象需要时间去初始化。比如，一个拥有50到100个Hibernate映射文件的项目可能花费10到20秒的时间去载这些映射文件，在运行每一个测试工具中的测试用例之前都会引发开销并导致总体测试运行变缓慢，降低开发效率。

测试类通常声明一批XML的资源路径或者Groovy的配置元数据——通常在类路径中——或者是一批用于配置应用程序的注解类。这些路径或者类跟在`web.xml`或者其它用于生产部署的配置文件中指定的是一样的。

通常，一旦被加载过一次，`ApplicationContext`就将被用于每个测试中。因此启动开销在一次测试集中将只会引发一次，随后执行的测试将会快得多。在这里，“测试集”的意思是在同一个JVM的所有测试——比如说，对给定项目或者模块的一次Ant、Maven或者Gradle构建运行的所有测试。在不太可能的情况下，一个测试会破坏应用上下文并引起重新加载——比如，修改一个bean定义或者应用程序对象的状态——TestContex框架将被设置为在开始下个测试之前重新加载配置并重建应用上下文。

参见[Section 11.5.4, “Context management”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#testcontext-ctx-management)和TestContext框架的[the section called “Context caching”](#)一节。

### 11.2.2 **测试配置的信赖注入**

当TestContext框架加载你的应用程序上下文的时候，它将通过信赖注入有选择性地配置测试实例。这为使用你的应用程序上下文中的预配置bean来建立测试配置提供了一个很方便的机制。这里有一个很大的便处就是你可以在不同的测试场景中重复使用应用程序上下文（比如，配置基于Spring管理的对象图、事务代理、数据源等等），这样省去了为每个测试用例建立复杂的测试配置的必要。

举例说明，考虑这样一个场景，我们有一个类叫做`HibernateTitleRepository`，它实现了`Title`领域实体的数据访问逻辑。我们想编写集成测试来测试以下方面：

* Spring配置：总的来说，就是与`HibernateTitleRepository`配置有关的一切是否正确和存在？

* Hibernate映射文件：是否所有映射都正确，并且延迟加载的设置是否准备就绪？

* `HibernateTitleRepository`的逻辑：此类中的配置实例是否与预期一致？

查看使用[TestContext框架](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/integration-testing.html#testcontext-fixture-di)进行测试配置的信赖注入。

### 11.2.3 **事务管理**

测试中访问一个真实数据库的一个常见的问题是在持久层存储状态付出的努力。即使你使用开发环境的数据库，改变相应的状态也会影响将来的测试。并且，许多操作——插入或者改变持久层数据——也不能在事务之外执行（或者验证）。

TestContext框架解决了这个问题。默认行为下，这个框架将为每个测试创建并回滚一个事务。你只需简单的假定事务是存在的并写你的代码即可。如果你调用事务代理对象，他们也会根据它们配置的语义正确执行。而且，如果一个测试方法在运行相应事务时删除了选定表中的内容，事务默认情况下会进行回滚，数据库会回到测试执行前的那个状态。事务通过定义在应用程序上下文中的`PlatformTransactionManager` bean来得到支持。

如果你需要提交事务——通常不会这样做，但有时当你想用一个特定的测试来填充或者修改数据库时也会显得有用——TestContext框架将根据[`@Commit`](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#integration-testing-annotations)注解的指示对事务进行提交而不是回滚。

查看使用[TestContext框架](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/integration-testing.html#testcontext-fixture-di)进行事务管理。

### 11.2.4 **集成测试的支持类**

Spring TestContext框架提供了一些支持来简化集成测试的编写。这些基础类为测试框架提供了定义良好的钩子，还有一些便利的实例变量和方法，使你能够访问：

* `ApplicationContext`，用于从整体上来进行显示的bean查找或者测试上下文的状态。
* `JdbcTemplate`，用于执行SQL语句来查询数据库。这些的查询可用于确认执行数据库相关的应用程序代码前后数据库的状态，并且Spring保证这些查询与应用程序代码在同一个事务作用域中执行。如果需要与ORM工具协同使用，请确保避免误报。

还有，你可能想用特定于你的项目的实例和方法来创建你自己自定义的，应用程序范围的超类。

查看[TestContext框架](http://docs.spring.io/spring/docs/5.0.0.BUILD-SNAPSHOT/spring-framework-reference/htmlsingle/#testcontext-support-classes)的支持类。

