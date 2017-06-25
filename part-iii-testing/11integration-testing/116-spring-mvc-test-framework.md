The_Spring MVC Test framework_provides first class support for testing Spring MVC code using a fluent API that can be used with JUnit, TestNG, or any other testing framework. It’s built on the[Servlet API mock objects](http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/mock/web/package-summary.html)from the`spring-test`module and hence does_not_use a running Servlet container. It uses the`DispatcherServlet`to provide full Spring MVC runtime behavior and provides support for loading actual Spring configuration with the_TestContext framework_in addition to a standalone mode in which controllers may be instantiated manually and tested one at a time.

_Spring MVC Test_also provides client-side support for testing code that uses the`RestTemplate`. Client-side tests mock the server responses and also do_not_use a running server.

| ![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/tip.png.pagespeed.ce.w22Wv-tZ37.png "\[Tip\]") |
| :--- |
| Spring Boot provides an option to write full, end-to-end integration tests that include a running server. If this is your goal please have a look at the[Spring Boot reference page](http://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-testing.html#boot-features-testing-spring-boot-applications). For more information on the differences between out-of-container and end-to-end integration tests, see[the section called “Differences between Out-of-Container and End-to-End Integration Tests”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/integration-testing.html#spring-mvc-test-vs-end-to-end-integration-tests). |



