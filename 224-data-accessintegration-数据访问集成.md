### 2.2.4 Data Access/Integration 数据访问/集成

The *Data Access/Integration* layer consists of the JDBC, ORM, OXM, JMS, and Transaction modules.

The `spring-jdbc` module provides a [JDBC](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/#jdbc-introduction)-abstraction layer that removes the need to do tedious JDBC coding and parsing of database-vendor specific error codes.

The `spring-tx` module supports [programmatic and declarative transaction](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/#transaction) management for classes that implement special interfaces and for *all your POJOs (Plain Old Java Objects)*.

The `spring-orm` module provides integration layers for popular [object-relational mapping](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/#orm-introduction) APIs, including [JPA](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/#orm-jpa) and [Hibernate](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/#orm-hibernate). Using the `spring-orm` module you can use these O/R-mapping frameworks in combination with all of the other features Spring offers, such as the simple declarative transaction management feature mentioned previously.

The `spring-oxm` module provides an abstraction layer that supports [Object/XML mapping](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/#oxm) implementations such as JAXB, Castor, JiBX and XStream.

The `spring-jms` module ([Java Messaging Service](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/#jms)) contains features for producing and consuming messages. Since Spring Framework 4.1, it provides integration with the `spring-messaging` module.

数据访问/集成层由`JDBC`，`ORM`，`OXM`，`JMS`和`Transaction `模块组成。

`spring-jdbc`模块提供了一个`JDBC`抽象层，消除了对繁琐的`JDBC`编码和解析数据库供应商特定的错误代码的需要。

`spring-tx`模块支持实现特殊接口的类以及所有`POJO`（普通`Java`对象）的编程和声明事务管理。

`spring-orm`模块为流行的对象关系映射`API`提供集成层，包括`JPA`和`Hibernate`。使用`spring-orm`模块，您可以使用这些`O / R`映射框架结合`Spring`提供的所有其他功能，例如前面提到的简单声明式事务管理功能。

`spring-oxm`模块提供了一个支持`对象/ XML`映射实现的抽象层，例如JAXB，Castor，JiBX和XStream。

`spring-jms`模块（Java消息服务）`包含用于生成和使用消息的功能`。从`Spring Framework 4.1`开始，它提供了与`spring-messaging`模块的集成。