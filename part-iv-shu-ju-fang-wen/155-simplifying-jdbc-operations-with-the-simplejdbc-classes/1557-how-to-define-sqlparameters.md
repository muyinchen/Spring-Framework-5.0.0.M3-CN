### 15.5.7**如何定义SqlParameters**

如何定义SimpleJdbc类和RDBMS操作类的参数，详见15.6： “[像Java对象那样操作JDBC](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/jdbc.html#jdbc-object)”,你需要使用`SqlParameter`或者是它的子类。通常需要在构造器中定义参数名和SQL类型。SQL类型使用`java.sql.Types`常量来定义。我们已经看到过类似于如下的定义：

```java
new SqlParameter("in_id", Types.NUMERIC),
    new SqlOutParameter("out_first_name", Types.VARCHAR),
```

上面第一行`SqlParameter` 定义了一个传入参数。IN参数可以同时在存储过程调用和`SqlQuery`查询中使用，它的子类在下面的章节也有覆盖。

上面第二行`SqlOutParameter`定义了在一次存储过程调用中使用的返回参数。还有一个`SqlInOutParameter`类，可以用于输入输出参数。也就是说，它既是一个传入参数，也是一个返回值。

| ![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| :--- |
| 备注：参数只有被定义成`SqlParameter`和`SqlInOutParameter`才可以提供输入值。不像`StoredProcedure`类为了考虑向后兼容允许定义为`SqlOutParameter`的参数可以提供输入值  |

For IN parameters, in addition to the name and the SQL type, you can specify a scale for numeric data or a type name for custom database types. For`out`parameters, you can provide a`RowMapper`to handle mapping of rows returned from a`REF`cursor. Another option is to specify an`SqlReturnType`that provides an opportunity to define customized handling of the return values.

对于输入参数，除了名字和SQL类型，你可以定义数值区间或是自定义数据类型名。针对输出参数，你可以使用RowMapper处理从REF游标返回的行映射。另外一种选择是定义SqlReturnType，可以针对返回值作自定义处理。

