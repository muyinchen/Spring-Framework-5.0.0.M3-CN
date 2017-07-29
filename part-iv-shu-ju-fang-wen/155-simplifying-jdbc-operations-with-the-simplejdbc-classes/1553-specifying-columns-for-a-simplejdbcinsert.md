### 15.5.3Specifying columns for a SimpleJdbcInsert

You can limit the columns for an insert by specifying a list of column names with the`usingColumns`method:

```java
public class JdbcActorDao implements ActorDao {

	private JdbcTemplate jdbcTemplate;
	private SimpleJdbcInsert insertActor;

	public void setDataSource(DataSource dataSource) {
		this.jdbcTemplate = new JdbcTemplate(dataSource);
		this.insertActor = new SimpleJdbcInsert(dataSource)
				.withTableName("t_actor")
				.usingColumns("first_name", "last_name")
				.usingGeneratedKeyColumns("id");
	}

	public void add(Actor actor) {
		Map<String, Object> parameters = new HashMap<String, Object>(2);
		parameters.put("first_name", actor.getFirstName());
		parameters.put("last_name", actor.getLastName());
		Number newId = insertActor.executeAndReturnKey(parameters);
		actor.setId(newId.longValue());
	}

	// ... additional methods

}
```

The execution of the insert is the same as if you had relied on the metadata to determine which columns to use.

