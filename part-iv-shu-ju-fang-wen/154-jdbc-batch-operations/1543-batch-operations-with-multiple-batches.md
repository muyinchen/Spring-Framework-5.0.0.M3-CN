### 15.4.3**多个批处理操作**

上面最后一个例子更新的批处理数量太大，最好能再分割成更小的块。最简单的方式就是你多次调用`batchUpdate`来实现，但是可以有更优的方法。要使用这个方法除了SQL语句，还需要传入参数集合对象，每次Batch的更新数和一个`ParameterizedPreparedStatementSetter`去设置预编译SQL语句的参数值。框架会循环调用提供的值并且将更新操作切割成指定数量的小批次。

下面的例子设置了更新批次数量为100的批量更新操作：

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

这个调用的批量更新方法返回一个包含int数组的二维数组，包含每次更新生效的行数。第一层数组长度代表批处理执行的数量，第二层数组长度代表每个批处理生效的更新数。每个批处理的更新数必须和所有批处理的大小匹配，除非是最后一次批处理可能小于这个数，具体依赖于更新对象的总数。每次更新语句生效的更新数由JDBC驱动提供。如果更新数量不存在，JDBC驱动会返回-2

