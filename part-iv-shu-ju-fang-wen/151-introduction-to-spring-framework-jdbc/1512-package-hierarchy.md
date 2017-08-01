### 15.1.2**包层级**

Spring的JDBC框架一共包含4种不同类型的包、包括`core`,`datasource`,`object`和`support`.

`org.springframework.jdbc.core`包含`JdbcTemplate` 类和它各种回调接口、外加一些相关的类。它的一个子包  
`org.springframework.jdbc.core.simple`包含`SimpleJdbcInsert`和`SimpleJdbcCall`等类。另一个叫`org.springframework.jdbc.core.namedparam`的子包包含`NamedParameterJdbcTemplate`及它的一些工具类。详见：  
15.2：“[使用JDBC核心类控制基础的JDBC处理过程和异常处理机制](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/jdbc.html#jdbc-core)”，15.4：“[JDBC批量操作](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/jdbc.html#jdbc-advanced-jdbc)”，15.5：“[利用SimpleJdbc 类简化JDBC操作](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/jdbc.html#jdbc-simple-jdbc)”.

`org.springframework.jdbc.datasource`包包含`DataSource`数据源访问的工具类，以及一些简单的`DataSource`实现用于测试和脱离JavaEE容器运行的JDBC代码。子包`org.springfamework.jdbc.datasource.embedded`提供Java内置数据库例如HSQL, H2, 和Derby的支持。详见：15.3：“[控制数据库连接](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/jdbc.html#jdbc-connections)”，15.8：“[内置数据库支持](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/jdbc.html#jdbc-embedded-database-support)”.

`org.springframework.jdbc.object`包含用于在RDBMS查询、更新和存储过程中创建线程安全及可重用的对象类。详见15.6： “[像Java对象那样操作JDBC](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/jdbc.html#jdbc-object)”；这种方式类似于JDO的查询方式，不过查询返回的对象是与数据库脱离的。此包针对JDBC做了很多上层封装、而底层依赖于`org.springframework.jdbc.core`包。

`org.springframework.jdbc.support`包含`SQLException`的转换类和一些工具类。JDBC处理过程中抛出的异常会被转换成`org.springframework.dao`里面定义的异常类。这意味着SpringJDBC抽象层的代码不需要实现JDBC或者RDBMS特定的错误处理方式。所有转换的异常都没有被捕获，而是让开发者自己处理异常、具体的话既可以捕获异常也可以直接抛给上层调用者详见：15.2.3：“[SQL异常转换器](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/jdbc.html#jdbc-SQLExceptionTranslator)”.

