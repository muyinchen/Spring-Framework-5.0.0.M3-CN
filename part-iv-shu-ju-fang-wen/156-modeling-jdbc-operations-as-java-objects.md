## 15.6 Modeling JDBC operations as Java objects

The`org.springframework.jdbc.object`package contains classes that allow you to access the database in a more object-oriented manner. As an example, you can execute queries and get the results back as a list containing business objects with the relational column data mapped to the properties of the business object. You can also execute stored procedures and run update, delete, and insert statements.

| ![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png.pagespeed.ce.9zQ_1wVwzR.png "\[Note\]") |
| :--- |
| Many Spring developers believe that the various RDBMS operation classes described below \(with the exception of the[`StoredProcedure`](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/jdbc.html#jdbc-StoredProcedure)class\) can often be replaced with straight`JdbcTemplate`calls. Often it is simpler to write a DAO method that simply calls a method on a`JdbcTemplate`directly \(as opposed to encapsulating a query as a full-blown class\).However, if you are getting measurable value from using the RDBMS operation classes, continue using these classes. |



