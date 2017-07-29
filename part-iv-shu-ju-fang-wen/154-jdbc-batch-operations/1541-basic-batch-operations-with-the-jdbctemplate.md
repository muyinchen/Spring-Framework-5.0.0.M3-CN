### 15.4.1Basic batch operations with the JdbcTemplate

You accomplish`JdbcTemplate`batch processing by implementing two methods of a special interface,`BatchPreparedStatementSetter`, and passing that in as the second parameter in your`batchUpdate`method call. Use the`getBatchSize`method to provide the size of the current batch. Use the`setValues`method to set the values for the parameters of the prepared statement. This method will be called the number of times that you specified in the`getBatchSize`call. The following example updates the actor table based on entries in a list. The entire list is used as the batch in this example:

```java
public class JdbcActorDao implements ActorDao {
	private JdbcTemplate jdbcTemplate;

	public void setDataSource(DataSource dataSource) {
		this.jdbcTemplate = new JdbcTemplate(dataSource);
	}

	public int[] batchUpdate(final List<Actor> actors) {
		int[] updateCounts = jdbcTemplate.batchUpdate("update t_actor set first_name = ?, " +
				"last_name = ? where id = ?",
			new BatchPreparedStatementSetter() {
				public void setValues(PreparedStatement ps, int i) throws SQLException {
						ps.setString(1, actors.get(i).getFirstName());
						ps.setString(2, actors.get(i).getLastName());
						ps.setLong(3, actors.get(i).getId().longValue());
					}

					public int getBatchSize() {
						return actors.size();
					}
				});
		return updateCounts;
	}

	// ... additional methods
}
```

If you are processing a stream of updates or reading from a file, then you might have a preferred batch size, but the last batch might not have that number of entries. In this case you can use the`InterruptibleBatchPreparedStatementSetter`interface, which allows you to interrupt a batch once the input source is exhausted. The`isBatchExhausted`method allows you to signal the end of the batch.

