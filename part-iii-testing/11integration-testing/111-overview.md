## 11. **集成测试**

## 11.1 **概述**

能够在不需要部署到应用服务器或连接到其它企业基础服务的前提下做一些集成测试是很重要的。这将使你能够测试以下内容：

* Spring IoC容器上下文的正确装配。

* 使用JDBC或其它ORM工具访问数据。这将包括SQL语句、Hibernate查询和JPA实体映射的正确性等等这些内容。

Spring Framework在`spring-test`模块中为集成测试提供了强有力的支持。该Jar包的实际名字可能会包含发布版本号而且可能是`org.springframework.test`这样长的形式，这取决于你是从哪获得的（请参阅[section on Dependency Management](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/overview.html#dependency-management)中的解释）。这个库包括了`org.springframework.test`包，其中包含了使用Spring容器进行集成测试的重要的类。这个测试不信赖于应用服务器和其它的发布环境。这些测试会比单元测试稍慢但比同类型的Selenium测试或信赖于发布到应用服务器的远程测试要快得多。

在Spring2.5及之后的版本，单元和集成测试支持是以注解驱动[Spring TestContext框架](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/integration-testing.html#testcontext-framework)这样的形式提供的。TestContext框架对实际使用的测试框架是不可知的的，因此可以使用包括 JUnit, TestNG等等许多测试手段。

