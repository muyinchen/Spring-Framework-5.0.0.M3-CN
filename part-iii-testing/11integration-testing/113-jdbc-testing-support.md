## 11.3 **JDBC测试支持**

_Note that AbstractTransactionalJUnit4SpringContextTests and AbstractTransactionalTestNGSpringContextTests provide convenience methods which delegate to the aforementioned methods in JdbcTestUtils._

The `spring-jdbc` module provides support for configuring and launching an embedded database which can be used in integration tests that interact with a database. For details, see [Section 15.8, “Embedded database support”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#jdbc-embedded-database-support) and [Section 15.8.5, “Testing data access logic with an embedded database”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#jdbc-embedded-database-dao-testing).

`org.springframework.test.jdbc`是包含`JdbcTestUtils`的包，它是一个JDBC相关的工具方法集，意在简化标准数据库测试场景。特别地，`JdbcTestUtils`提供以下静态工具方法：

* `countRowsInTable(..)`：统计给定表的行数。
* `countRowsInTableWhere(..)`：使用提供的`where`语句进行筛选统计给定表的行数。
* `deleteFromTables(..)`：删除特定表的全部数据。
* `deleteFromTableWhere(..)`：使用提供的`where`语句进行筛选并删除给定表的数据。
* `dropTables(..)`：删除指定的表。

注意[_`AbstractTransactionalJUnit4SpringContextTests`_](http://docs.spring.io/spring/docs/5.0.0.BUILD-SNAPSHOT/spring-framework-reference/htmlsingle/#testcontext-support-classes-junit4)_`和`_[_`AbstractTransactionalTestNGSpringContextTests`_](http://docs.spring.io/spring/docs/5.0.0.BUILD-SNAPSHOT/spring-framework-reference/htmlsingle/#testcontext-support-classes-testng)`提供了委托给前面所述的JdbcTestUtils中的方法的简便方法。`

`spring-jdbc`模块提供了配置和启动嵌入式数据库的支持，可用于与数据库交互的集成测试中。

详见[Section 15.8, “嵌入式数据库支持”](http://docs.spring.io/spring/docs/5.0.0.BUILD-SNAPSHOT/spring-framework-reference/htmlsingle/#jdbc-embedded-database-support)和[ection 15.8.5, “使用嵌入式数据库测试数据访问逻辑”](http://docs.spring.io/spring/docs/5.0.0.BUILD-SNAPSHOT/spring-framework-reference/htmlsingle/#jdbc-embedded-database-dao-testing)。

