### 15.4.2Batch operations with a List of objects

Both the`JdbcTemplate`and the`NamedParameterJdbcTemplate`provides an alternate way of providing the batch update. Instead of implementing a special batch interface, you provide all parameter values in the call as a list. The framework loops over these values and uses an internal prepared statement setter. The API varies depending on whether you use named parameters. For the named parameters you provide an array of`SqlParameterSource`, one entry for each member of the batch. You can use the`SqlParameterSource.createBatch`method to create this array, passing in either an array of JavaBeans or an array of Maps containing the parameter values.

This example shows a batch update using named parameters:

```java
public class JdbcActorDao implements ActorDao {
	private NamedParameterTemplate namedParameterJdbcTemplate;

	public void setDataSource(DataSource dataSource) {
		this.namedParameterJdbcTemplate = new NamedParameterJdbcTemplate(dataSource);
	}

	public int[] batchUpdate(final List<Actor> actors) {
		SqlParameterSource[] batch = SqlParameterSourceUtils.createBatch(actors.toArray());
		int[] updateCounts = namedParameterJdbcTemplate.batchUpdate(
				"update t_actor set first_name = :firstName, last_name = :lastName where id = :id",
				batch);
		return updateCounts;
	}

	// ... additional methods
}
```

For an SQL statement using the classic "?" placeholders, you pass in a list containing an object array with the update values. This object array must have one entry for each placeholder in the SQL statement, and they must be in the same order as they are defined in the SQL statement.

The same example using classic JDBC "?" placeholders:

```java
public class JdbcActorDao implements ActorDao {

	private JdbcTemplate jdbcTemplate;

	public void setDataSource(DataSource dataSource) {
		this.jdbcTemplate = new JdbcTemplate(dataSource);
	}

	public int[] batchUpdate(final List<Actor> actors) {
		List<Object[]> batch = new ArrayList<Object[]>();
		for (Actor actor : actors) {
			Object[] values = new Object[] {
					actor.getFirstName(),
					actor.getLastName(),
					actor.getId()};
			batch.add(values);
		}
		int[] updateCounts = jdbcTemplate.batchUpdate(
				"update t_actor set first_name = ?, last_name = ? where id = ?",
				batch);
		return updateCounts;
	}

	// ... additional methods

}
```

All of the above batch update methods return an int array containing the number of affected rows for each batch entry. This count is reported by the JDBC driver. If the count is not available, the JDBC driver returns a -2 value.

