### 15.5.8Calling a stored function using SimpleJdbcCall

You call a stored function in almost the same way as you call a stored procedure, except that you provide a function name rather than a procedure name. You use the`withFunctionName`method as part of the configuration to indicate that we want to make a call to a function, and the corresponding string for a function call is generated. A specialized execute call,`executeFunction,`is used to execute the function and it returns the function return value as an object of a specified type, which means you do not have to retrieve the return value from the results map. A similar convenience method named`executeObject`is also available for stored procedures that only have one`out`parameter. The following example is based on a stored function named`get_actor_name`that returns an actor’s full name. Here is the MySQL source for this function:

```java
CREATE FUNCTION get_actor_name (in_id INTEGER)
RETURNS VARCHAR(200) READS SQL DATA
BEGIN
	DECLARE out_name VARCHAR(200);
	SELECT concat(first_name, ' ', last_name)
		INTO out_name
		FROM t_actor where id = in_id;
	RETURN out_name;
END;
```

To call this function we again create a`SimpleJdbcCall`in the initialization method.

```
public class JdbcActorDao implements ActorDao {

	private JdbcTemplate jdbcTemplate;
	private SimpleJdbcCall funcGetActorName;

	public void setDataSource(DataSource dataSource) {
		this.jdbcTemplate = new JdbcTemplate(dataSource);
		JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
		jdbcTemplate.setResultsMapCaseInsensitive(true);
		this.funcGetActorName = new SimpleJdbcCall(jdbcTemplate)
				.withFunctionName("get_actor_name");
	}

	public String getActorName(Long id) {
		SqlParameterSource in = new MapSqlParameterSource()
				.addValue("in_id", id);
		String name = funcGetActorName.executeFunction(String.class, in);
		return name;
	}

	// ... additional methods

}
```

The execute method used returns a`String`containing the return value from the function call.

