### 15.3.8 DataSourceTransactionManager

`DataSourceTransactionManager`类实现了`PlatformTransactionManager`接口。它将JDBC连接从指定的数据源绑定到当前执行的线程中，允许一个线程连接对应一个数据源。

应用代码需要通过`DataSourceUtils.getConnection(DataSource) `来获取JDBC连接，而不是通过JavaEE标准的`DataSource.getConnection`来获取。它会抛出`org.springframework.dao`的运行时异常而不是编译时SQL异常。所有框架类像`JdbcTemplate`都默认使用这个策略。如果不需要和这个 `DataSourceTransactionManager`类一起使用，`DataSourceUtils `提供的功能跟一般的数据库连接策略没有什么两样，因此它可以在任何场景下使用。

`DataSourceTransactionManager`支持自定义隔离级别，以及JDBC查询超时机制。为了支持后者，应用代码必须在每个创建的语句中使用`JdbcTemplate`或是调用`DataSourceUtils.applyTransactionTimeout(..)`方法

在单一的资源使用场景下它可以替代`JtaTransactionManager`，不需要要求容器去支持JTA。如果你严格遵循连接查找的模式的话、可以通过配置来做彼此切换。JTA本身不支持自定义隔离级别！

