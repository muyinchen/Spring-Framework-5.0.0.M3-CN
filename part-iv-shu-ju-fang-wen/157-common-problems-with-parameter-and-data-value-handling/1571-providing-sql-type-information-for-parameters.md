### 15.7.1**为参数设置SQL的类型信息**

通常Sping通过传入的参数类型决定SQL的参数类型。可以在设定参数值的时候显式提供SQL类型。有些场景下设置NULL值是有必要的。

你可以通过以下方式来设置SQL类型信息：

* 许多`JdbcTemplate`的更新和查询方法需要传入额外的`int`数组类型的参数。这个数组使用`java.sql.Types`类的常量值来确定相关参数的SQL类型。每个参数会有对应的类型项。

* 你可以使用`SqlParameterValue`类来包装需要额外信息的参数值。针对每个值创建一个新的实例，并且在构造函数中传入SQL类型和参数值。你还可以传入数值类型的可选区间参数

* 对于那些使用命名参数的情况，使用`SqlParameterSource`类型的类比如`BeanPropertySqlParameterSource `，或`MapSqlParameterSource`。他们都具备了为命名参数注册SQL类型的功能。







