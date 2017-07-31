### 15.1.1**选择一种JDBC数据库访问方法**

JDBC数据库访问有几种基本的途径可供选择。除了JdbcTemplate的三种使用方式外，新的SimpleJdbcInsert和SimplejdbcCall调用类通过优化数据库元数据（来简化JDBC操作），还有一种更偏向于面向对象的RDBMS对象风格的方法、有点类似于JDO的查询设计。即使你已经选择了其中一种方法、你仍然可以混合使用另外一种方法的某一个特性。所有的方法都需要JDBC2.0兼容驱动的支持，一些更高级的特性则需要使用JDBC3.0驱动支持。

* \_JdbcTemplate 是经典的Spring JDBC访问方式，也是最常用的。这是“最基础”的方式、其他所有方式都是在 JdbcTemplate的基础之上封装的。

* \_NamedParameterJdbcTemplate 在原有`JdbcTemplate`的基础上做了一层包装支持命名参数特性、用于替代传统的JDBC“？”占位符。当SQL语句中包含多个参数时使用这种方式能有更好的可读性和易用性

* \_SimpleJdbcInsert和SimpleJdbcCall操作类主要利用JDBC驱动所提供的数据库元数据的一些特性来简化数据库操作配置。这种方式简化了编码、你只需要提供表或者存储过程的名字、以及和列名相匹配的参数Map。但前提是数据库需要提供足够的元数据。如果数据库没有提供这些元数据，需要开发者显式配置参数的映射关系。

* \_RDBMS对象的方式包含MappingSqlQuery, SqlUpdate和StoredProcedure，需要你在初始化应用数据访问层时创建可重用和线程安全的对象。这种方式设计上类似于JDO查询、你可以定义查询字符串，声明参数及编译查询语句。一旦完成这些工作之后，执行方法可以根据不同的传入参数被多次调用。



