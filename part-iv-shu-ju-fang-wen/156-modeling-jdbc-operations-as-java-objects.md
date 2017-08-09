## 15.6 **像Java对象那样操作JDBC**

The`org.springframework.jdbc.object`package contains classes that allow you to access the database in a more object-oriented manner. As an example, you can execute queries and get the results back as a list containing business objects with the relational column data mapped to the properties of the business object. You can also execute stored procedures and run update, delete, and insert statements.

`org.springframework.jdbc.object`包能让你更加面向对象化的访问数据库。举个例子，用户可以执行查询并返回一个list， 该list作为一个结果集将把从数据库中取出的列数据映射到业务对象的属性上。你也可以执行存储过程，包括更新、删除、插入语句。

| ![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png.pagespeed.ce.9zQ_1wVwzR.png "\[Note\]") |
| :--- |
|  备注：许多Spring的开发者认为下面将描述的各种RDBMS操作类（[`StoredProcedure`](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/jdbc.html#jdbc-StoredProcedure)类除外）可以直接被`JdbcTemplate`代替； 相对于把一个查询操作封装成一个类而言，直接调用`JdbcTemplate`方法将更简单而且更容易理解。但这仅仅是一种观点而已， 如果你认为可以从直接使用RDBMS操作类中获取一些额外的好处，你不妨根据自己的需要和喜好进行不同的选择。. |



