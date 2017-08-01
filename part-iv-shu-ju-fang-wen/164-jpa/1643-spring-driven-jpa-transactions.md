### 16.4.3 Spring-driven JPA transactions

| ![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png.pagespeed.ce.9zQ_1wVwzR.png "\[Note\]") |
| :--- |
| You are_strongly_encouraged to read[Section 13.5, “Declarative transaction management”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/transaction.html#transaction-declarative)if you have not done so, to get a more detailed coverage of Spring’s declarative transaction support. |

The recommended strategy for JPA is local transactions via JPA’s native transaction support. Spring’s`JpaTransactionManager`provides many capabilities known from local JDBC transactions, such as transaction-specific isolation levels and resource-level read-only optimizations, against any regular JDBC connection pool \(no XA requirement\).

Spring JPA also allows a configured`JpaTransactionManager`to expose a JPA transaction to JDBC access code that accesses the same`DataSource`, provided that the registered`JpaDialect`supports retrieval of the underlying JDBC`Connection`. Out of the box, Spring provides dialects for the EclipseLink and Hibernate JPA implementations. See the next section for details on the`JpaDialect`mechanism.

