### 15.7.1Providing SQL type information for parameters

Usually Spring determines the SQL type of the parameters based on the type of parameter passed in. It is possible to explicitly provide the SQL type to be used when setting parameter values. This is sometimes necessary to correctly set NULL values.

You can provide SQL type information in several ways:

* Many update and query methods of the`JdbcTemplate`take an additional parameter in the form of an`int`array. This array is used to indicate the SQL type of the corresponding parameter using constant values from the`java.sql.Types`class. Provide one entry for each parameter.

* You can use the`SqlParameterValue`class to wrap the parameter value that needs this additional information.Create a new instance for each value and pass in the SQL type and parameter value in the constructor. You can also provide an optional scale parameter for numeric values.

* For methods working with named parameters, use the`SqlParameterSource`classes`BeanPropertySqlParameterSource`or`MapSqlParameterSource`. They both have methods for registering the SQL type for any of the named parameter values.



