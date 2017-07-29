### 15.4.3Batch operations with multiple batches

The last example of a batch update deals with batches that are so large that you want to break them up into several smaller batches. You can of course do this with the methods mentioned above by making multiple calls to the`batchUpdate`method, but there is now a more convenient method. This method takes, in addition to the SQL statement, a Collection of objects containing the parameters, the number of updates to make for each batch and a`ParameterizedPreparedStatementSetter`to set the values for the parameters of the prepared statement. The framework loops over the provided values and breaks the update calls into batches of the size specified.

This example shows a batch update using a batch size of 100:

```java
public class JdbcActorDao implements ActorDao {

	private JdbcTemplate jdbcTemplate;

	public void setDataSource(DataSource dataSource) {
		this.jdbcTemplate = new JdbcTemplate(dataSource);
	}

	public int[][] batchUpdate(final Collection<Actor> actors) {
		int[][] updateCounts = jdbcTemplate.batchUpdate(
				"update t_actor set first_name = ?, last_name = ? where id = ?",
				actors,
				100,
				new ParameterizedPreparedStatementSetter<Actor>() {
					public void setValues(PreparedStatement ps, Actor argument) throws SQLException {
						ps.setString(1, argument.getFirstName());
						ps.setString(2, argument.getLastName());
						ps.setLong(3, argument.getId().longValue());
					}
				});
		return updateCounts;
	}

	// ... additional methods

}

```

The batch update methods for this call returns an array of int arrays containing an array entry for each batch with an array of the number of affected rows for each update. The top level array’s length indicates the number of batches executed and the second level array’s length indicates the number of updates in that batch. The number of updates in each batch should be the batch size provided for all batches except for the last one that might be less, depending on the total number of update objects provided. The update count for each update statement is the one reported by the JDBC driver. If the count is not available, the JDBC driver returns a -2 value.

