### 15.5.2Retrieving auto-generated keys using SimpleJdbcInsert

This example uses the same insert as the preceding, but instead of passing in the id it retrieves the auto-generated key and sets it on the new Actor object. When you create the`SimpleJdbcInsert`, in addition to specifying the table name, you specify the name of the generated key column with the`usingGeneratedKeyColumns`method.

```java
public class JdbcActorDao implements ActorDao {

	private JdbcTemplate jdbcTemplate;
	private SimpleJdbcInsert insertActor;

	public void setDataSource(DataSource dataSource) {
		this.jdbcTemplate = new JdbcTemplate(dataSource);
		this.insertActor = new SimpleJdbcInsert(dataSource)
				.withTableName("t_actor")
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

The main difference when executing the insert by this second approach is that you do not add the id to the Map and you call the`executeAndReturnKey`method. This returns a`java.lang.Number`object with which you can create an instance of the numerical type that is used in our domain class. You cannot rely on all databases to return a specific Java class here;`java.lang.Number`is the base class that you can rely on. If you have multiple auto-generated columns, or the generated values are non-numeric, then you can use a`KeyHolder`that is returned from the`executeAndReturnKeyHolder`method.

