### 16.4.3 Spring驱动的JPA事务

| ![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png.pagespeed.ce.9zQ_1wVwzR.png "\[Note\]") |
| :--- |
| 如果开发者还没有阅读[Section 13.5, “声明式事务管理”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/transaction.html#transaction-declarative)，强烈建议开发者先行阅读，这样可以更详细地了解Spring的对声明式事务支持。 |

JPA的推荐策略是通过JPA的本地事务支持的本地事务。 Spring的`JpaTransactionManager`提供了许多来自本地JDBC事务的功能，例如针对任何常规JDBC连接池（不需要XA要求）指定事务的隔离级别和资源级只读优化等。

Spring JPA还允许配置`JpaTransactionManager`将JPA事务暴露给访问同一`DataSource`的JDBC访问代码，前提是注册的`JpaDialect`支持检索底层JDBC连接。Spring为EclipseLink和Hibernate JPA实现提供了实现。有关`JpaDialect`机制的详细信息，请参阅下一节。

