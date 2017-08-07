### 15.5.2**使用SimpleJdbcInsert获取自增Key**

接下来，我们对于同样的插入语句，我们并不传入id，而是通过数据库自动获取主键的方式来创建新的Actor对象并插入数据库。 当我们创建`SimpleJdbcInsert`实例时, 我们不仅需要指定表名，同时我们通过`usingGeneratedKeyColumns`方法指定需要数据库自动生成主键的列名。

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

执行插入操作时第二种方式最大的区别是你不是在Map中指定ID，而是调用`executeAndReturnKey`方法。这个方法返回`java.lang.Number`对象，可以创建一个数值类型的实例用于我们的领域模型中。你不能仅仅依赖所有的数据库都返回一个指定的Java类；`java.lang.Number`是你可以依赖的基础类。如果你有多个自增列，或者自增的值是非数值型的，你可以使用`executeAndReturnKeyHolder`方法返回的`KeyHolder`

