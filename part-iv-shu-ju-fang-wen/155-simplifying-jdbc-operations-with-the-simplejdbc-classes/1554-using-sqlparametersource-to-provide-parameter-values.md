### 15.5.4Using SqlParameterSource to provide parameter values

Using a`Map`to provide parameter values works fine, but it’s not the most convenient class to use. Spring provides a couple of implementations of the`SqlParameterSource`interface that can be used instead.The first one is`BeanPropertySqlParameterSource`, which is a very convenient class if you have a JavaBean-compliant class that contains your values. It will use the corresponding getter method to extract the parameter values. Here is an example:

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
		SqlParameterSource parameters = new BeanPropertySqlParameterSource(actor);
		Number newId = insertActor.executeAndReturnKey(parameters);
		actor.setId(newId.longValue());
	}

	// ... additional methods

}
```

Another option is the`MapSqlParameterSource`that resembles a Map but provides a more convenient`addValue`method that can be chained.

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
		SqlParameterSource parameters = new MapSqlParameterSource()
				.addValue("first_name", actor.getFirstName())
				.addValue("last_name", actor.getLastName());
		Number newId = insertActor.executeAndReturnKey(parameters);
		actor.setId(newId.longValue());
	}

	// ... additional methods

}
```

As you can see, the configuration is the same; only the executing code has to change to use these alternative input classes.

