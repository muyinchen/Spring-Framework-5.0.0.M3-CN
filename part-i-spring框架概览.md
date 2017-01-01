# Part I. Spring框架概览

The Spring Framework is a lightweight solution and a potential one-stop-shop for building your enterprise-ready applications. However, Spring is modular, allowing you to use only those parts that you need, without having to bring in the rest. You can use the IoC container, with any web framework on top, but you can also use only the[Hibernate integration code](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/#orm-hibernate) or the [JDBC abstraction layer](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/#jdbc-introduction). The Spring Framework supports declarative transaction management, remote access to your logic through RMI or web services, and various options for persisting your data. It offers a full-featured [MVC framework](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/#mvc-introduction), and enables you to integrate [AOP](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/#aop-introduction) transparently into your software.

`Spring Framework`是一种轻量级的解决方案，是构建你的企业级应用程序的潜在一站式解决方案。 尽管如此，`Spring`是模块化的，允许你只使用你需要的那些部分，而不必引入其他的。 你可以使用IoC容器，任何`Web`框架在顶部(只是底层用Spring框架，比如ssh，中间那层用了`Spring`)，但你也可以只使用`Hibernate`集成代码或`JDBC`抽象层。 `Spring`框架支持声明式事务管理，通过RMI或Web服务远程访问你的逻辑，以及用于持久存储数据的各种选项。 它提供了一个全功能的`MVC`框架，并使你能够将`AOP`透明地集成到你的软件中。

Spring is designed to be non-intrusive, meaning that your domain logic code generally has no dependencies on the framework itself. In your integration layer (such as the data access layer), some dependencies on the data access technology and the Spring libraries will exist. However, it should be easy to isolate these dependencies from the rest of your code base.

`Spring`设计为非侵入式的，这意味着你所写的逻辑代码通常没有对框架本身的依赖。 在你的集中处理层（例如数据访问层）中，将存在对数据访问技术和Spring库的一些依赖。 但是，应该很容易将这些依赖关系与其余代码库隔离开。

This document is a reference guide to Spring Framework features. If you have any requests, comments, or questions on this document, please post them on the [user mailing list](https://groups.google.com/forum/#!forum/spring-framework-contrib). Questions on the Framework itself should be asked on StackOverflow (see [https://spring.io/questions](https://spring.io/questions)).

本文档是`Spring Framework`特性的参考指南。 如果你对本文档有任何要求，意见或问题，请将其张贴在用户邮件列表中。 框架本身的问题应该在`StackOverflow`（请参阅https://spring.io/questions）。



