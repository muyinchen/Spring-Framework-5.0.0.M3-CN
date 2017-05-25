## 10. 单元测试

比起传统的 Java EE 开发方式，依赖注入可以弱化你的代码对容器的依赖。在基于 Junit 或 TestNG 的测试代码中，无需依赖于 Spring 或其他容器，你只需通过 `new` 操作符，便可以创建出组成你的应用程序的各种 POJO 对象。而通过 [mock 对象](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#mock-objects)（以及其它各种测试技术的综合运用），你可以将被测试的代码单独隔离开来进行测试。如果你在进行架构设计时遵循了 Spring 所推荐的模式，那么由此带来的诸如清晰的分层、组件化等等优点也会使你更容易地对你的代码进行单元测试。例如，通过 mock DAO 层或 Repositry 接口的实现，针对服务层对象的单元测试代码并不需要真正地访问持久层的数据。

真正的单元测试代码运行的非常快，因为并不需要一整套的运行时架构去支持测试的运转。所以在开发中强调真正的单元测试必须成为编码方法论中的一部分将会极大地提高你的生产力。

也许没有这一节内容的帮助你也能写出高效的针对 IoC 应用的单元测试，但是以下将要提到的 Spring 所提供的 mock 对象和测试支持类，在一些特定的场景中会对单元测试很有帮助。

## 10.1 Mock 对象

### 10.1.1 环境

`org.springframework.mock.env` 包含了对抽象 `Environment` 和 `PropertySource` 的 Mock 实现（参考 [Section 3.13.1 1Bean的定义文件](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#beans-definition-profiles) 和 [Section 3.13.3 PropertySource抽象](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#beans-property-source-abstraction)）。`MockEnvironment` 和 `MockPropertySource` 对于编写针对依赖于环境相关属性的代码的，与容器无关的测试用例很有帮助。

### 10.1.2 JNDI

`org.springframework.mock.jndi` 包含了 JNDI SPI 的实现。这一实现可以让你为你的测试套件或独立应用配置起一个简单的 JNDI 环境。例如，如果测试代码和 Java EE 容器的 JDBC `DataSource`都绑定到同一个 JNDI 名字上，你就可以在测试中直接复用应用程序的代码和配置而不需做任何改动。

### 10.1.3 Servlet API

The `org.springframework.mock.web` package contains a comprehensive set of Servlet API mock objects, which are useful for testing web contexts, controllers, and filters. These mock objects are targeted at usage with Spring’s Web MVC framework and are generally more convenient to use than dynamic mock objects such as [EasyMock](http://www.easymock.org/) or alternative Servlet API mock objects such as [MockObjects](http://www.mockobjects.com/). Since Spring Framework 4.0, the set of mocks in the `org.springframework.mock.web`package is based on the Servlet 3.0 API.

For thorough integration testing of your Spring MVC and REST `Controller`s in conjunction with your `WebApplicationContext` configuration for Spring MVC, see the[_Spring MVC Test Framework_](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#spring-mvc-test-framework).

`org.springframework.mock.web` 包含了十分全面的 Servlet API mock 对象。这些对象在测试web 上下文，Controller 和 Filter 的时候很有用。由于这些 mock 对象是有针对性地为了与 Spring 的 Web MVC 框架共同使用而编写的，因此相比起诸如 EasyMock 这种动态mock 对象或 MockObjects 这种替代性的 Servlet API mock 对象，使用起来要更为方便。

假如是要把 Spring MVC 和 REST `Controller`与 `WebApplicationContext` 配置结合起来进行一番彻底的集成测试，请使用 [_Spring MVC 测试框架_](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#spring-mvc-test-framework) 。

## 10.2 单元测试支持类

### 10.2.1 通用支持工具

`org.springframework.test.util` 包含了一些供单元测试和集成测试中使用的通用工具。

`ReflectionTestUtils` 是一组基于反射的方法集合。在测试包含如下一些测试用例的应用时，开发人员可以使用这些工具方法应对诸如更改一个常量的值，赋值一个非公有字段，调用一个非公有setter方法，或调用一个非公有的 配置 或 生命周期 回调方法等测试场景：

* 诸如 JPA 和 Hibernate 等广泛采用`private`或`protected`访问方式而`public` setter方法来访问domain entity属性的ORM（Object-Relational Mapping\)框架
* `@Autowired`，`@Inject`和`@Resource`等用于对`private`或`protected`字段，setter 方法和配置方法进行依赖注入的 Spring 注解。
* `@PostConstruct`和`@PreDestroy`等用在生命周期回调方法上的注解。

`AopTestUtils`是一组 AOP 相关工具方法的集合。这些方法可以用于帮助获取隐藏于一重或多重 Spring 代理背后目标对象的引用。举个栗子，你使用 EasyMock 或 Mockito mock 了一个被包装在 Spring 代理之中的 bean，这时你可能需要对这个 mock 进行直接的访问，从而能够配置对此 mock 的期望行为并在稍后执行验证。关于 Spring 提供的核心 AOP 工具，请参考`AopUtils`和`AopProxyUtils`这两个类。

### 10.2.2 Spring MVC

The `org.springframework.test.web` package contains `ModelAndViewAssert`, which you can use in combination with JUnit, TestNG, or any other testing framework for unit tests dealing with Spring MVC `ModelAndView` objects.

`org.springframework.test.web`包含了`ModelAndViewAssert`类。你可以在用 Junit，TestNG 或任何测试框架编写的单元测试中使用这个类来帮助你跟 Spring MVC 框架的`ModelAndView`对象进行互动。

| ![\[Tip\]](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/images/tip.png.pagespeed.ce.w22Wv-tZ37.png) |
| --- |
| 注：如果想要像测试 POJO 一样来测试你的 Spring MVC `Controller`，你可以将`ModelAndViewAssert`与`MockHttpServletRequest`、`MockHttpSession`等来自 [Servlet API mocks](#) 的 Mock 类结合使用。而假如是要把 Spring MVC 和 REST `Controller` 与`WebApplicationContext`配置结合起来进行一番彻底的集成测试，请使用[_Spring MVC 测试框架_](#)。 |
|  |



