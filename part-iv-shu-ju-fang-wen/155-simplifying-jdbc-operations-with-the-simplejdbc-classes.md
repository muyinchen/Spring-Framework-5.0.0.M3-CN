## 15.5 **利用SimpleJdbc类简化JDBC操作**

`SimpleJdbcInsert`类和`SimpleJdbcCall`类主要利用了JDBC驱动所提供的数据库元数据的一些特性来简化数据库操作配置。这意味着可以在前端减少配置，当然你也可以覆盖或是关闭底层的元数据处理，在代码里面指定所有的细节。

