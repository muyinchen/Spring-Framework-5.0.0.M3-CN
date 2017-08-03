### 15.3.3 SmartDataSource

实现`SmartDataSource`接口的实现类需要能够提供到关系数据库的连接。它继承了`DataSource`接口，允许使用它的类查询是否在某个特定的操作后需要关闭连接。这在当你需要重用连接时比较有用。

