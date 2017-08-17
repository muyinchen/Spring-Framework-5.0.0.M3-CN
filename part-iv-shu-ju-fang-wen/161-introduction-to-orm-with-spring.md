## 16.1介绍一下Spring中的ORM

Spring框架在实现资源管理、数据访问对象（DAO）层，和事务策略等方面，支持对Java持久化API（JPA）以及原生Hibernate的集成。以Hibernate举例来说，Spring有非常赞的IoC功能，可以解决许多典型的Hibernate配置和集成问题。开发者可以通过依赖注入来配置O-R（对象关系）映射组件支持的特性。Hibernate的这些特性可以参与Spring的资源和事务管理，并且符合Spring的通用事务和DAO层的异常体系。因此，Spring团队推荐开发者使用Spring集成的方式来开发DAO层，而不是使用原生的Hibernate或者JPA的API。老版本的Spring DAO模板现在不推荐使用了，想了解这部分内容可以参考[经典ORM使用](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/classic-spring.html#classic-spring-orm)一节。

当开发者创建数据访问应用程序时，Spring会为开发者选择的ORM层对应功能进行优化。而且，开发者可以根据需要来利用Spring对集成ORM的支持，开发者应该将此集成工作与维护内部类似的基础架构服务的成本和风险进行权衡。同时，开发者在使用Spring集成的时候可以很大程度上不用考虑技术，将ORM的支持当做一个库来使用，因为所有的组件都被设计为可重用的JavaBean组件了。Spring IoC容器中的ORM十分易于配置和部署。本节中的大多数示例都是讲解在Spring容器中的来如何配置。

开发者使用Spring框架来中创建自己的ORM DAO的好处如下：

* \__易于测试_. \_Spring IoC的模式使得开发者可以轻易的替换Hibernate的`SessionFactory`实例，JDBC的`DataSource`实例，事务管理器，以及映射对象（如果有必要）的配置和实现。这一特点十分利于开发者对每个模块进行独立的测试。

* \__泛化数据访问异常_.\_  Spring可以将ORM工具的异常封装起来，将所有异常（可以是受检异常）封装成运行时的`DataAccessException`体系。这一特性可以令开发者在合适的逻辑层上处理绝大多数不可修复的持久化异常，避免了大量的`catch`,`throw`和异常的声明。开发者还可以按需来处理这些异常。其中，JDBC异常（包括一些特定DB语言）都会被封装为相同的体系，意味着开发者即使使用不同的JDBC操作，基于不同的DB，也可以保证一致的编程模型。

* \__通用的资源管理_.\_  Spring的应用上下文可以通过处理配置源的位置来灵活配置Hibernate的`SessionFactory`实例，JPA的`EntityManagerFactory`实例，JDBC的`DataSource`实例以及其他类似的资源。Spring的这一特性使得这些实例的配置十分易于管理和修改。同时，Spring还为处理持久化资源的配置提供了高效，易用和安全的处理方式。举个例子，有些代码使用了Hibernate需要使用相同的`Session`来确保高效性和正确的事务处理。Spring通过Hibernate的`SessionFactory`来获取当前的`Session`，来透明的将`Session`绑定到当前的线程。Srping为任何本地或者JTA事务环境解决了在使用Hibernate时碰到的一些常见问题。

* \__集成事务管理_.\_  开发者可以通过`@Transactional`注解或在XML配置文件中显式配置事务AOP Advise拦截，将ORM代码封装在声明式的AOP方法拦截器中。事务的语义和异常处理（回滚等）都可以根据开发者自己的需求来定制。在后面的章节中，[Resource and transaction management](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/orm.html#orm-resource-mngmnt)中，开发者可以在不影响ORM相关代码的情况下替换使用不同的事务管理器。例如，开发者可以在本地事务和JTA之间进行交换，并在两种情况下具有相同的完整服务（如声明式事务）。而且，JDBC相关的代码在事务上完全和处理ORM部分的代码集成。这对于不适用于ORM的数据访问非常有用，例如批处理和BLOB流式传输，仍然需要与ORM操作共享常见事务。

| ![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/tip.png.pagespeed.ce.w22Wv-tZ37.png) |
| :--- |
| 为了更全面的ORM支持，包括支持其他类型的数据库技术（如MongoDB），开发者可能需要查看[Spring Data](http://projects.spring.io/spring-data/)系列项目。如果开发者是JPA用户，则可以从[https://spring.io](https://spring.io/)的查阅[开始使用JPA访问数据指南](https://spring.io/guides/gs/accessing-data-jpa/)一文进行简单了解。 |



