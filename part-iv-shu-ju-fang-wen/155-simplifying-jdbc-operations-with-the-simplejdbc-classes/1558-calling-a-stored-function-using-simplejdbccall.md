### 15.5.8**使用SimpleJdbcCall调用内置存储函数**

调用存储函数几乎和调用存储过程的方式是一样的，唯一的区别你提供的是函数名而不是存储过程名。你可以使用`withFunctionName`方法作为配置的一部分表示我们想要调用一个函数，以及生成函数调用相关的字符串。一个特殊的execute调用，`executeFunction`，用来指定这个函数并且返回一个指定类型的函数值，这意味着你不需要从结果Map获取返回值。存储过程也有一个名字为`executeObject`的便捷方法，但是只要一个输出参数。下面的例子基于一个名字为`get_actor_name`的存储函数，返回actor的全名。下面是这个函数的Mysql源代码：

```java
CREATE FUNCTION get_actor_name (in_id INTEGER)
RETURNS VARCHAR(200) READS SQL DATA
BEGIN
    DECLARE out_name VARCHAR(200);
    SELECT concat(first_name, ' ', last_name)
        INTO out_name
        FROM t_actor where id = in_id;
    RETURN out_name;
END;
```

我们需要在初始方法中创建`SimpleJdbcCall`来调用这个函数

```java
public class JdbcActorDao implements ActorDao {

    private JdbcTemplate jdbcTemplate;
    private SimpleJdbcCall funcGetActorName;

    public void setDataSource(DataSource dataSource) {
        this.jdbcTemplate = new JdbcTemplate(dataSource);
        JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
        jdbcTemplate.setResultsMapCaseInsensitive(true);
        this.funcGetActorName = new SimpleJdbcCall(jdbcTemplate)
                .withFunctionName("get_actor_name");
    }

    public String getActorName(Long id) {
        SqlParameterSource in = new MapSqlParameterSource()
                .addValue("in_id", id);
        String name = funcGetActorName.executeFunction(String.class, in);
        return name;
    }

    // ... additional methods

}
```

execute方法返回一个包含函数调用返回值的字符串

