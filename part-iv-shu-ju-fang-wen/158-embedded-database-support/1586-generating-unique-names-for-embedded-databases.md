### 15.8.6**生成内嵌数据库的唯一名字**

开发团队在使用内嵌数据库时经常碰到的一个错误是：当他们的测试集想对同一个数据库创建额外的实例。这种错误在以下场景经常发生，XML配置文件或者`@Configuration`类用于创建内嵌数据库，并且相关的配置在同样的测试集的多个测试场景下都被用到（例如，在同一个JVM进程中）。例如，针对内嵌数据库的不同集成测试的`ApplicationContext`配置的区别只在当前哪个Bean定义是有效的。

这些错误的根源是Spring的`EmbeddedDatabaseFactory`工厂（ XML命名空间和Java Config对象的`EmbeddedDatabaseBuilder`内部都用到了这个）会将内嵌数据库的名字默认设置成”testdb”.针对的场景，内嵌数据库通常设置成和Bean `Id`相同的名字。（例如，常用像“`dataSource`”的名字）。结果，接下来创建内嵌数据库的尝试都没创建一个新的数据库。相反，同样的JDBC链接URL被重用。创建内嵌数据的库的尝试往往从同一个配置返回了已存在的内嵌数据库实例。

为了解决这个问题Spring框架4.2 提供了生成内嵌数据库唯一名的支持。例如：

* `EmbeddedDatabaseFactory.setGenerateUniqueDatabaseName()`

* `EmbeddedDatabaseBuilder.generateUniqueName()`

* `<jdbc:embedded-database generate-name="true" …​ >`



