### 15.3.7 TransactionAwareDataSourceProxy

`TransactionAwareDataSourceProxy`is a proxy for a target`DataSource`, which wraps that target`DataSource`to add awareness of Spring-managed transactions. In this respect, it is similar to a transactional JNDI`DataSource`as provided by a Java EE server.

| ![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png.pagespeed.ce.9zQ_1wVwzR.png "\[Note\]") |
| :--- |
| It is rarely desirable to use this class, except when already existing code that must be called and passed a standard JDBC`DataSource`interface implementation. In this case, it’s possible to still have this code be usable, and at the same time have this code participating in Spring managed transactions. It is generally preferable to write your own new code using the higher level abstractions for resource management, such as`JdbcTemplate`or`DataSourceUtils`. |

_\(See the`TransactionAwareDataSourceProxy`javadocs for more details.\)_

