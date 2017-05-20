## 11.2 Goals of Integration Testing

Spring’s integration testing support has the following primary goals:

- To manage [Spring IoC container caching](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#testing-ctx-management) between test execution.
- To provide [Dependency Injection of test fixture instances](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#testing-fixture-di).
- To provide [transaction management](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#testing-tx) appropriate to integration testing.
- To supply [Spring-specific base classes](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#testing-support-classes) that assist developers in writing integration tests.

The next few sections describe each goal and provide links to implementation and configuration details.

### 11.2.1 Context management and caching

The Spring TestContext Framework provides consistent loading of Spring `ApplicationContext`s and `WebApplicationContext`s as well as caching of those contexts. Support for the caching of loaded contexts is important, because startup time can become an issue — not because of the overhead of Spring itself, but because the objects instantiated by the Spring container take time to instantiate. For example, a project with 50 to 100 Hibernate mapping files might take 10 to 20 seconds to load the mapping files, and incurring that cost before running every test in every test fixture leads to slower overall test runs that reduce developer productivity.

Test classes typically declare either an array of *resource locations* for XML or Groovy configuration metadata — often in the classpath — or an array of *annotated classes*that is used to configure the application. These locations or classes are the same as or similar to those specified in `web.xml` or other configuration files for production deployments.

By default, once loaded, the configured `ApplicationContext` is reused for each test. Thus the setup cost is incurred only once per test suite, and subsequent test execution is much faster. In this context, the term *test suite* means all tests run in the same JVM — for example, all tests run from an Ant, Maven, or Gradle build for a given project or module. In the unlikely case that a test corrupts the application context and requires reloading — for example, by modifying a bean definition or the state of an application object — the TestContext framework can be configured to reload the configuration and rebuild the application context before executing the next test.

See [Section 11.5.4, “Context management”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#testcontext-ctx-management) and [the section called “Context caching”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#testcontext-ctx-management-caching) with the TestContext framework.

### 11.2.2 Dependency Injection of test fixtures

When the TestContext framework loads your application context, it can optionally configure instances of your test classes via Dependency Injection. This provides a convenient mechanism for setting up test fixtures using preconfigured beans from your application context. A strong benefit here is that you can reuse application contexts across various testing scenarios (e.g., for configuring Spring-managed object graphs, transactional proxies, `DataSource`s, etc.), thus avoiding the need to duplicate complex test fixture setup for individual test cases.

As an example, consider the scenario where we have a class, `HibernateTitleRepository`, that implements data access logic for a `Title` domain entity. We want to write integration tests that test the following areas:

- The Spring configuration: basically, is everything related to the configuration of the `HibernateTitleRepository` bean correct and present?
- The Hibernate mapping file configuration: is everything mapped correctly, and are the correct lazy-loading settings in place?
- The logic of the `HibernateTitleRepository`: does the configured instance of this class perform as anticipated?

See dependency injection of test fixtures with the [TestContext framework](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#testcontext-fixture-di).

### 11.2.3 Transaction management

One common issue in tests that access a real database is their effect on the state of the persistence store. Even when you’re using a development database, changes to the state may affect future tests. Also, many operations — such as inserting or modifying persistent data — cannot be performed (or verified) outside a transaction.

The TestContext framework addresses this issue. By default, the framework will create and roll back a transaction for each test. You simply write code that can assume the existence of a transaction. If you call transactionally proxied objects in your tests, they will behave correctly, according to their configured transactional semantics. In addition, if a test method deletes the contents of selected tables while running within the transaction managed for the test, the transaction will roll back by default, and the database will return to its state prior to execution of the test. Transactional support is provided to a test via a `PlatformTransactionManager` bean defined in the test’s application context.

If you want a transaction to commit — unusual, but occasionally useful when you want a particular test to populate or modify the database — the TestContext framework can be instructed to cause the transaction to commit instead of roll back via the [`@Commit`](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#integration-testing-annotations) annotation.

See transaction management with the [TestContext framework](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#testcontext-tx).

### 11.2.4 Support classes for integration testing

The Spring TestContext Framework provides several `abstract` support classes that simplify the writing of integration tests. These base test classes provide well-defined hooks into the testing framework as well as convenient instance variables and methods, which enable you to access:

- The `ApplicationContext`, for performing explicit bean lookups or testing the state of the context as a whole.
- A `JdbcTemplate`, for executing SQL statements to query the database. Such queries can be used to confirm database state both *prior to* and *after* execution of database-related application code, and Spring ensures that such queries run in the scope of the same transaction as the application code. When used in conjunction with an ORM tool, be sure to avoid [false positives](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#testcontext-tx-false-positives).

In addition, you may want to create your own custom, application-wide superclass with instance variables and methods specific to your project.

See support classes for the [TestContext framework](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#testcontext-support-classes).