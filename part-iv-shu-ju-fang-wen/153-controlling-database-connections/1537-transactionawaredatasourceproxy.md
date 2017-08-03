### 15.3.7 TransactionAwareDataSourceProxy

`TransactionAwareDataSourceProxy`会创建一个目标`DataSource`的代理，内部包装了`DataSource`，在此基础上添加了Spring事务管理功能。有点类似于JavaEE服务器中提供的JNDI事务数据源。

| ![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png.pagespeed.ce.9zQ_1wVwzR.png "\[Note\]") |
| :--- |
| 注意：一般情况下很少用到这个类，除非现有代码在被调用的时候需要一个标准的 JDBC `DataSource`接口实现作为参数。在这种场景下，使用proxy可以仍旧重用老代码，同时能够有Spring管理事务的能力。更多的场景下更推荐使用`JdbcTemplate`和`DataSourceUtils`等更高抽象的资源管理类 . |

（更多细节请查看`TransactionAwareDataSourceProxy`的JavaDoc）

