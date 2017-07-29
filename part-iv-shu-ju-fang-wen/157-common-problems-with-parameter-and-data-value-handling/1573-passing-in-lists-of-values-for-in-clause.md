### 15.7.3Passing in lists of values for IN clause

The SQL standard allows for selecting rows based on an expression that includes a variable list of values. A typical example would be`select * from T_ACTOR where id in (1, 2, 3)`. This variable list is not directly supported for prepared statements by the JDBC standard; you cannot declare a variable number of placeholders. You need a number of variations with the desired number of placeholders prepared, or you need to generate the SQL string dynamically once you know how many placeholders are required. The named parameter support provided in the`NamedParameterJdbcTemplate`and`JdbcTemplate`takes the latter approach. Pass in the values as a`java.util.List`of primitive objects. This list will be used to insert the required placeholders and pass in the values during the statement execution.

| ![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| :--- |
| Be careful when passing in many values. The JDBC standard does not guarantee that you can use more than 100 values for an`in`expression list. Various databases exceed this number, but they usually have a hard limit for how many values are allowed. Oracle’s limit is 1000. |

In addition to the primitive values in the value list, you can create a`java.util.List`of object arrays. This list would support multiple expressions defined for the`in`clause such as`select * from T_ACTOR where (id, last_name) in ((1, 'Johnson'), (2, 'Harrop'\))`. This of course requires that your database supports this syntax.

