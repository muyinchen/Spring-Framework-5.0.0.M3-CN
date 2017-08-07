### 15.5.4**使用SqlParameterSource 提供参数值**

使用`Map`来指定参数值没有问题，但不是最便捷的方法。Spring提供了一些`SqlParameterSource`接口的实现类来更方便的做这些操作。第一个是`BeanPropertySqlParameterSource`，如果你有一个JavaBean兼容的类包含具体的值，使用这个类是很方便的。他会使用相关的Getter方法来获取参数值。下面是一个例子：

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

另外一个选择是使用`MapSqlParameterSource`，类似于Map、但是提供了一个更便捷的`addValue`方法可以用来做链式操作。

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

上面这些例子可以看出、配置是一样的，区别只是切换了不同的提供参数的实现方式来执行调用。

