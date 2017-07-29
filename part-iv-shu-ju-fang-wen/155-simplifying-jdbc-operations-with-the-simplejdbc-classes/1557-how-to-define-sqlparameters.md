### 15.5.7How to define SqlParameters

To define a parameter for the SimpleJdbc classes and also for the RDBMS operations classes, covered in[Section15.6, “Modeling JDBC operations as Java objects”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/jdbc.html#jdbc-object), you use an`SqlParameter`or one of its subclasses. You typically specify the parameter name and SQL type in the constructor. The SQL type is specified using the`java.sql.Types`constants. We have already seen declarations like:

```java
new SqlParameter("in_id", Types.NUMERIC),
	new SqlOutParameter("out_first_name", Types.VARCHAR),
```

The first line with the`SqlParameter`declares an IN parameter. IN parameters can be used for both stored procedure calls and for queries using the`SqlQuery`and its subclasses covered in the following section.

The second line with the`SqlOutParameter`declares an`out`parameter to be used in a stored procedure call. There is also an`SqlInOutParameter`for`InOut`parameters, parameters that provide an`IN`value to the procedure and that also return a value.

| ![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| :--- |
| Only parameters declared as`SqlParameter`and`SqlInOutParameter`will be used to provide input values. This is different from the`StoredProcedure`class, which for backwards compatibility reasons allows input values to be provided for parameters declared as`SqlOutParameter`. |

For IN parameters, in addition to the name and the SQL type, you can specify a scale for numeric data or a type name for custom database types. For`out`parameters, you can provide a`RowMapper`to handle mapping of rows returned from a`REF`cursor. Another option is to specify an`SqlReturnType`that provides an opportunity to define customized handling of the return values.

