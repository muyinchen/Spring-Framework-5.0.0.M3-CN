### 15.4.1**使用JdbcTemplate来进行基础的批量操作**

通过`JdbcTemplate `实现批处理需要实现特定接口的两个方法，`BatchPreparedStatementSetter`，并且将其作为第二个参数传入到`batchUpdate`方法调用中。使用`getBatchSize`提供当前批量操作的大小。使用`setValues`方法设置语句的Value参数。这个方法会按`getBatchSize`设置中指定的调用次数。下面的例子中通过传入列表来批量更新actor表。在这个例子中整个列表使用了批量操作：

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

如果你需要处理批量更新或者从文件中批量读取，你可能需要确定一个合适的批处理大小，但是最后一次批处理可能达不到这个大小。在这种场景下你可以使用`InterruptibleBatchPreparedStatementSetter`接口，允许在输入流耗尽之后终止批处理，`isBatchExhausted`方法使得你可以指定批处理结束时间。

