### 15.7.3传入IN语句的列表值

SQL标准允许基于一个带参数列表的表达式进行查询，一个典型的例子是`select * from T_ACTOR where id in (1, 2, 3)`. 这样的可变参数列表没有被JDBC标准直接支持；你不能定义可变数量的占位符（placeholder），只能定义固定变量的占位符，或者你在动态生成SQL字符串的时候需要提前知晓所需占位符的数量。`NamedParameterJdbcTemplate` 和 `JdbcTemplate `都使用了后面那种方式。当你传入参数时，你需要传入一个`java.util.List`类型，支持基本类型。而这个list将会在SQL执行时替换占位符并传入参数。

| ![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| :--- |
| 备注：在传入多个值的时候需要注意。JDBC标准不保证你能在一个`in`表达式列表中传入超过100个值，不少数据库会超过这个值，但是一般都会有个上限。比如Oracle的上限是1000. |

除了值列表的元数据值，你可以创建`java.util.List`的对象数组。这个列表支持多个在`in`语句内定义的表达式例如`select * from T_ACTOR where (id, last_name) in ((1, ‘Johnson’), (2, ‘Harrop’\))`。当然有个前提你的数据库需要支持这个语法。

