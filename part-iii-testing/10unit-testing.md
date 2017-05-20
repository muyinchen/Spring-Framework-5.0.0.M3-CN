## 10. Unit Testing

Dependency Injection should make your code less dependent on the container than it would be with traditional Java EE development. The POJOs that make up your application should be testable in JUnit or TestNG tests, with objects simply instantiated using the `new` operator, *without Spring or any other container*. You can use [mock objects](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#mock-objects) (in conjunction with other valuable testing techniques) to test your code in isolation. If you follow the architecture recommendations for Spring, the resulting clean layering and componentization of your codebase will facilitate easier unit testing. For example, you can test service layer objects by stubbing or mocking DAO or Repository interfaces, without needing to access persistent data while running unit tests.

True unit tests typically run extremely quickly, as there is no runtime infrastructure to set up. Emphasizing true unit tests as part of your development methodology will boost your productivity. You may not need this section of the testing chapter to help you write effective unit tests for your IoC-based applications. For certain unit testing scenarios, however, the Spring Framework provides the following mock objects and testing support classes.

## 10.1 Mock Objects

### 10.1.1 Environment

The `org.springframework.mock.env` package contains mock implementations of the `Environment` and `PropertySource` abstractions (see [Section 3.13.1, “Bean definition profiles”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#beans-definition-profiles) and [Section 3.13.3, “PropertySource abstraction”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#beans-property-source-abstraction)). `MockEnvironment` and `MockPropertySource` are useful for developing *out-of-container* tests for code that depends on environment-specific properties.

### 10.1.2 JNDI

The `org.springframework.mock.jndi` package contains an implementation of the JNDI SPI, which you can use to set up a simple JNDI environment for test suites or stand-alone applications. If, for example, JDBC `DataSource`s get bound to the same JNDI names in test code as within a Java EE container, you can reuse both application code and configuration in testing scenarios without modification.

### 10.1.3 Servlet API

The `org.springframework.mock.web` package contains a comprehensive set of Servlet API mock objects, which are useful for testing web contexts, controllers, and filters. These mock objects are targeted at usage with Spring’s Web MVC framework and are generally more convenient to use than dynamic mock objects such as [EasyMock](http://www.easymock.org/) or alternative Servlet API mock objects such as [MockObjects](http://www.mockobjects.com/). Since Spring Framework 4.0, the set of mocks in the `org.springframework.mock.web`package is based on the Servlet 3.0 API.

For thorough integration testing of your Spring MVC and REST `Controller`s in conjunction with your `WebApplicationContext` configuration for Spring MVC, see the[*Spring MVC Test Framework*](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#spring-mvc-test-framework).

## 10.2 Unit Testing support Classes

### 10.2.1 General testing utilities

The `org.springframework.test.util` package contains several general purpose utilities for use in unit and integration testing.

`ReflectionTestUtils` is a collection of reflection-based utility methods. Developers use these methods in testing scenarios where they need to change the value of a constant, set a non-`public` field, invoke a non-`public` setter method, or invoke a non-`public` *configuration* or *lifecycle* callback method when testing application code involving use cases such as the following.

- ORM frameworks such as JPA and Hibernate that condone `private` or `protected` field access as opposed to `public` setter methods for properties in a domain entity.
- Spring’s support for annotations such as `@Autowired`, `@Inject`, and `@Resource`, which provides dependency injection for `private` or `protected` fields, setter methods, and configuration methods.
- Use of annotations such as `@PostConstruct` and `@PreDestroy` for lifecycle callback methods.

`AopTestUtils` is a collection of AOP-related utility methods. These methods can be used to obtain a reference to the underlying target object hidden behind one or more Spring proxies. For example, if you have configured a bean as a dynamic mock using a library like EasyMock or Mockito and the mock is wrapped in a Spring proxy, you may need direct access to the underlying mock in order to configure expectations on it and perform verifications. For Spring’s core AOP utilities, see `AopUtils` and `AopProxyUtils`.

### 10.2.2 Spring MVC

The `org.springframework.test.web` package contains `ModelAndViewAssert`, which you can use in combination with JUnit, TestNG, or any other testing framework for unit tests dealing with Spring MVC `ModelAndView` objects.

| ![[Tip]](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/images/tip.png.pagespeed.ce.w22Wv-tZ37.png) |
| ---------------------------------------- |
| To unit test your Spring MVC `Controller`s as POJOs, use `ModelAndViewAssert` combined with `MockHttpServletRequest`, `MockHttpSession`, and so on from Spring’s [Servlet API mocks](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#mock-objects-servlet). For thorough integration testing of your Spring MVC and REST `Controller`s in conjunction with your `WebApplicationContext` configuration for Spring MVC, use the [*Spring MVC Test Framework*](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#spring-mvc-test-framework) instead. |