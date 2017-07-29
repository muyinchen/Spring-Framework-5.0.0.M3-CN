### 15.5.1Inserting data using SimpleJdbcInsert

Let’s start by looking at the`SimpleJdbcInsert`class with the minimal amount of configuration options. You should instantiate the`SimpleJdbcInsert`in the data access layer’s initialization method. For this example, the initializing method is the`setDataSource`method. You do not need to subclass the`SimpleJdbcInsert`class; simply create a new instance and set the table name using the`withTableName`method. Configuration methods for this class follow the "fluid" style that returns the instance of the`SimpleJdbcInsert`, which allows you to chain all configuration methods. This example uses only one configuration method; you will see examples of multiple ones later.

```java
public class JdbcActorDao implements ActorDao {

	private JdbcTemplate jdbcTemplate;
	private SimpleJdbcInsert insertActor;

	public void setDataSource(DataSource dataSource) {
		this.jdbcTemplate = new JdbcTemplate(dataSource);
		this.insertActor = new SimpleJdbcInsert(dataSource).withTableName("t_actor");
	}

	public void add(Actor actor) {
		Map<String, Object> parameters = new HashMap<String, Object>(3);
		parameters.put("id", actor.getId());
		parameters.put("first_name", actor.getFirstName());
		parameters.put("last_name", actor.getLastName());
		insertActor.execute(parameters);
	}

	// ... additional methods
}
```

The execute method used here takes a plain`java.utils.Map`as its only parameter. The important thing to note here is that the keys used for the Map must match the column names of the table as defined in the database. This is because we read the metadata in order to construct the actual insert statement.

