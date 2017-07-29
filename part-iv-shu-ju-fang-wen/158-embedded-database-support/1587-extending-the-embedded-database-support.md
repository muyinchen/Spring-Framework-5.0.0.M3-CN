### 15.8.7Extending the embedded database support

Spring JDBC embedded database support can be extended in two ways:

* Implement`EmbeddedDatabaseConfigurer`to support a new embedded database type.

* Implement`DataSourceFactory`to support a new`DataSource`implementation, such as a connection pool to manage embedded database connections.

You are encouraged to contribute back extensions to the Spring community at[jira.spring.io](https://jira.spring.io/browse/SPR).

