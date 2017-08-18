### 16.2.1资源和事务管理

通常企业应用都会包含很多重复的的资源管理代码。很多项目总是尝试去创造自己的解决方案，有时会为了开发的方便而牺牲对错误的处理。Spring为资源的配置管理提供了简单易用的解决方案，在JDBC上使用模板技术，在ORM上使用AOP拦截技术。

Spring的基础设施提供了合适的资源处理，同时Spring引入了DAO层的异常体系，可以适用于任何数据访问策略。对于JDBC直连来说，前面提及到的`JdbcTemplate`类提供了包括连接处理，对`SQLException`到`DataAccessException`的异常封装，同时还包含对于一些特定数据库SQL错误代码的转换。对于ORM技术来说，可以参考下一节来了解异常封装的优点。

当谈到事务管理时，`JdbcTemplate`类通过Spring事务管理器挂接到Spring事务支持，并支持JTA和JDBC事务。Spring通过Hibernate，JPA事务管理器和JTA的支持来提供Hibernate和JPA这类ORM技术的支持。想了解更多关于事务的描述，可以参考第13章，[事务管理](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/transaction.html)。

