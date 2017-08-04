### 15.4.2**对象列表的批量处理**

`JdbcTemplate`和`NamedParameterJdbcTemplate`都提供了批量更新的替代方案。这个时候不是实现一个特定的批量接口，而是在调用时传入所有的值列表。框架会循环访问这些值并且使用内部的SQL语句setter方法。你是否已声明参数对应API是不一样的。针对已声明参数你需要传入qlParameterSource数组，每项对应单次的批量操作。你可以使用`SqlParameterSource.createBatch`方法来创建这个数组，传入JavaBean数组或是包含参数值的Map数组。

下面是一个使用已声明参数的批量更新例子：

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

对于使用“？”占位符的SQL语句，你需要传入带有更新值的对象数组。对象数组每一项对应SQL语句中的一个占位符，并且传入顺序需要和SQL语句中定义的顺序保持一致。

下面是使用经典JDBC“？”占位符的例子：

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

上面所有的批量更新方法都返回一个数组，包含具体成功的行数。这个计数是由JDBC驱动返回的。如果拿不到计数。JDBC驱动会返回-2。

