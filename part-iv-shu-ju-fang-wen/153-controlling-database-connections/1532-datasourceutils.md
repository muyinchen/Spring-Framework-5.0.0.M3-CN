### 15.3.2 DataSourceUtils

`DataSourceUtils`类是一个方便有用的工具类，提供了从JNDI获取和关闭连接等有用的静态方法。它支持线程绑定的连接、例如：使用`DataSourceTransactionManager`的时候，将把数据库连接绑定到当前的线程上。

