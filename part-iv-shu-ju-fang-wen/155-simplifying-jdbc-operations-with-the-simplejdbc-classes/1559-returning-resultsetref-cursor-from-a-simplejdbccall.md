### 15.5.9Returning ResultSet/REF Cursor from a SimpleJdbcCall

Calling a stored procedure or function that returns a result set is a bit tricky. Some databases return result sets during the JDBC results processing while others require an explicitly registered`out`parameter of a specific type. Both approaches need additional processing to loop over the result set and process the returned rows. With the`SimpleJdbcCall`you use the`returningResultSet`method and declare a`RowMapper`implementation to be used for a specific parameter. In the case where the result set is returned during the results processing, there are no names defined, so the returned results will have to match the order in which you declare the`RowMapper`implementations. The name specified is still used to store the processed list of results in the results map that is returned from the execute statement.

The next example uses a stored procedure that takes no IN parameters and returns all rows from the t\_actor table. Here is the MySQL source for this procedure:

```java
CREATE PROCEDURE read_all_actors()
BEGIN
 SELECT a.id, a.first_name, a.last_name, a.birth_date FROM t_actor a;
END;
```

To call this procedure you declare the`RowMapper`. Because the class you want to map to follows the JavaBean rules, you can use a`BeanPropertyRowMapper`that is created by passing in the required class to map to in the`newInstance`method.

```java
public class JdbcActorDao implements ActorDao {

	private SimpleJdbcCall procReadAllActors;

	public void setDataSource(DataSource dataSource) {
		JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
		jdbcTemplate.setResultsMapCaseInsensitive(true);
		this.procReadAllActors = new SimpleJdbcCall(jdbcTemplate)
				.withProcedureName("read_all_actors")
				.returningResultSet("actors",
				BeanPropertyRowMapper.newInstance(Actor.class));
	}

	public List getActorsList() {
		Map m = procReadAllActors.execute(new HashMap<String, Object>(0));
		return (List) m.get("actors");
	}

	// ... additional methods

}
```

The execute call passes in an empty Map because this call does not take any parameters. The list of Actors is then retrieved from the results map and returned to the caller.

