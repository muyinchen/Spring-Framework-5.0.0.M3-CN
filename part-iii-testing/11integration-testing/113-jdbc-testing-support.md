## 11.3 JDBC Testing Support

The `org.springframework.test.jdbc` package contains `JdbcTestUtils`, which is a collection of JDBC related utility functions intended to simplify standard database testing scenarios. Specifically, `JdbcTestUtils` provides the following static utility methods.

- `countRowsInTable(..)`: counts the number of rows in the given table
- `countRowsInTableWhere(..)`: counts the number of rows in the given table, using the provided `WHERE` clause
- `deleteFromTables(..)`: deletes all rows from the specified tables
- `deleteFromTableWhere(..)`: deletes rows from the given table, using the provided `WHERE` clause
- `dropTables(..)`: drops the specified tables

*Note that AbstractTransactionalJUnit4SpringContextTests and AbstractTransactionalTestNGSpringContextTests provide convenience methods which delegate to the aforementioned methods in JdbcTestUtils.*

The `spring-jdbc` module provides support for configuring and launching an embedded database which can be used in integration tests that interact with a database. For details, see [Section 15.8, “Embedded database support”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#jdbc-embedded-database-support) and [Section 15.8.5, “Testing data access logic with an embedded database”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#jdbc-embedded-database-dao-testing).