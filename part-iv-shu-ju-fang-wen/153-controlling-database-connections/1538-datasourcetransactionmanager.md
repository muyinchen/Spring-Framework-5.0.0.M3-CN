### 15.3.8 DataSourceTransactionManager

The`DataSourceTransactionManager`class is a`PlatformTransactionManager`implementation for single JDBC datasources. It binds a JDBC connection from the specified data source to the currently executing thread, potentially allowing for one thread connection per data source.

Application code is required to retrieve the JDBC connection through`DataSourceUtils.getConnection(DataSource)`instead of Java EE’s standard`DataSource.getConnection`. It throws unchecked`org.springframework.dao`exceptions instead of checked`SQLExceptions`. All framework classes like`JdbcTemplate`use this strategy implicitly. If not used with this transaction manager, the lookup strategy behaves exactly like the common one - it can thus be used in any case.

The`DataSourceTransactionManager`class supports custom isolation levels, and timeouts that get applied as appropriate JDBC statement query timeouts. To support the latter, application code must either use`JdbcTemplate`or call the`DataSourceUtils.applyTransactionTimeout(..)`method for each created statement.

This implementation can be used instead of`JtaTransactionManager`in the single resource case, as it does not require the container to support JTA. Switching between both is just a matter of configuration, if you stick to the required connection lookup pattern. JTA does not support custom isolation levels!

