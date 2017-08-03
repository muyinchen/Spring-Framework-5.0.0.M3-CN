### 15.3.6 DriverManagerDataSource



`DriverManagerDataSource`类实现了标准的`DataSource`接口，可以通过Java Bean属性来配置原生的JDBC驱动，并且每次都返回一个新的连接。

这个实现对于测试和JavaEE容器以外的独立环境比较有用，无论是作为一个在Spring IOC容器内的`DataSource `Bean，或是在单一的JNDI环境中。由于`Connection.close()`仅仅只是简单的关闭数据库连接，因此任何能够操作`DataSource`的持久层代码都能很好的工作。但是，使用JavaBean类型的连接池，比如`commons-dbcp`往往更简单、即使是在测试环境下也是如此，因此更推荐`commons-dbcp`。

