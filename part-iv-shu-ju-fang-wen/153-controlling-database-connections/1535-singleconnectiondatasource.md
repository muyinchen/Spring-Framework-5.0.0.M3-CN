### 15.3.5 SingleConnectionDataSource

`SingleConnectionDataSource`实现了`SmartDataSource`接口、内部封装了一个在每次使用后都不会关闭的单一连接。显然，这种场景下无法支持多线程。

为了防止客户端代码误以为数据库连接来自连接池（就像使用持久化工具时一样）错误的调用`close`方法，你应将`suppressClose`设置为`true`。这样，通过该类获取的将是代理连接（禁止关闭）而不是原有的物理连接。需要注意你不能将这个类强制转换成Oracle等数据库的原生连接。

这个类主要用于测试目的。例如，他使得测试代码能够脱离应用服务器，很方便的在单一的JNDI环境下调试。和`DriverManagerDataSource`相反，它总是重用相同的连接，这是为了避免在测试过程中创建过多的物理连接。

