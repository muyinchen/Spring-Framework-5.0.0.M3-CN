### 15.5.5**利用SimpleJdbcCall调用存储过程**

`SimpleJdbcCall`利用数据库元数据的特性来查找传入的参数和返回值，这样你就不需要显式去定义他们。如果你喜欢的话也以自己定义参数，尤其对于某些参数，你无法直接将他们映射到Java类上，例如`ARRAY`类型和`STRUCT`类型的参数。下面第一个例子展示了一个存储过程，从一个MySQL数据库返回`Varchar`和`Date`类型。这个存储过程例子从指定的actor记录中查询返回`first_name`,`last_name`,和`birth_date`列。

```java
CREATE PROCEDURE read_actor (
    IN in_id INTEGER,
    OUT out_first_name VARCHAR(100),
    OUT out_last_name VARCHAR(100),
    OUT out_birth_date DATE)
BEGIN
    SELECT first_name, last_name, birth_date
    INTO out_first_name, out_last_name, out_birth_date
    FROM t_actor where id = in_id;
END;
```

`in_id` 参数包含你正在查找的actor记录的`id`参数返回从数据库表读取的数据

`SimpleJdbcCall `和`SimpleJdbcInsert`定义的方式比较类似。你需要在数据访问层的初始化代码中初始化和配置该类。相比`StoredProcedure`类，你不需要创建一个子类并且不需要定义能够在数据库元数据中查找到的参数。下面是一个使用上面存储过程的`SimpleJdbcCall`配置例子。除了`DataSource`以外唯一的配置选项是存储过程的名字

```java
public class JdbcActorDao implements ActorDao {

    private JdbcTemplate jdbcTemplate;
    private SimpleJdbcCall procReadActor;

    public void setDataSource(DataSource dataSource) {
        this.jdbcTemplate = new JdbcTemplate(dataSource);
        this.procReadActor = new SimpleJdbcCall(dataSource)
                .withProcedureName("read_actor");
    }

    public Actor readActor(Long id) {
        SqlParameterSource in = new MapSqlParameterSource()
                .addValue("in_id", id);
        Map out = procReadActor.execute(in);
        Actor actor = new Actor();
        actor.setId(id);
        actor.setFirstName((String) out.get("out_first_name"));
        actor.setLastName((String) out.get("out_last_name"));
        actor.setBirthDate((Date) out.get("out_birth_date"));
        return actor;
    }

    // ... additional methods

}
```

调用代码包括创建包含传入参数的`SqlParameterSource`。这里需要重视的是传入参数值名字需要和存储过程中定义的参数名称相匹配。有一种场景不需要匹配、那就是你使用元数据去确定数据库对象如何与存储过程相关联。在存储过程源代码中指定的并不一定是数据库中存储的格式。有些数据库会把名字转成大写、而另外一些会使用小写或者特定的格式。

`execute`方法接受传入参数，同时返回一个`Map`包含任意的返回参数，Map的Key是存储过程中指定的名字。在这个例子中它们是`out_first_name`, `out_last_name` 和 `out_birth_date`

`execute `方法的最后一部分使用返回的数据创建Actor对象实例。再次需要强调的是`Out`参数的名字必须是存储过程中定义的。结果Map中存储的返回参数名必须和数据库中的返回参数名（不同的数据库可能会不一样）相匹配，为了提高你代码的可重用性，你需要在查找中区分大小写，或者使用Spring里面的`LinkedCaseInsensitiveMap`。如果使用`LinkedCaseInsensitiveMap`，你需要创建自己的`JdbcTemplate`并且将`setResultsMapCaseInsensitive`属性设置为`True`。然后你将自定义的`JdbcTemplate `传入到`SimpleJdbcCall`的构造器中。下面是这种配置的一个例子：

```java
public class JdbcActorDao implements ActorDao {

    private SimpleJdbcCall procReadActor;

    public void setDataSource(DataSource dataSource) {
        JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
        jdbcTemplate.setResultsMapCaseInsensitive(true);
        this.procReadActor = new SimpleJdbcCall(jdbcTemplate)
                .withProcedureName("read_actor");
    }

    // ... additional methods

}
```

通过这样的配置，你就可以无需担心`out`参数值的大小写问题。

