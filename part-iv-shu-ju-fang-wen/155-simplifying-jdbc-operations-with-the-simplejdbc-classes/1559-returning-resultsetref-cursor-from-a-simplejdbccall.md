### 15.5.9**从SimpleJdbcCall返回ResultSet/REF游标**

调用存储过程或者函数返回结果集会相对棘手一点。一些数据库会在JDBC结果处理中返回结果集，而另外一些数据库则需要明确指定返回值的类型。两种方式都需要循环迭代结果集做额外处理。通过`SimpleJdbcCall`，你可以使用`returningResultSet`方法，并定义一个`RowMapper`的实现类来处理特定的返回值。 当结果集在返回结果处理过程中没有被定义名称时，返回的结果集必须与定义的`RowMapper`的实现类指定的顺序保持一致。 而指定的名字也会被用作返回结果集中的名称。

下面的例子使用了一个不包含输入参数的存储过程并且返回t\_actor标的所有行。下面是这个存储过程的Mysql源代码：

```java
CREATE PROCEDURE read_all_actors()
BEGIN
 SELECT a.id, a.first_name, a.last_name, a.birth_date FROM t_actor a;
END;
```

调用这个存储过程你需要定义`RowMapper`。因为我们定义的Map类遵循JavaBean规范，所以我们可以使用`BeanPropertyRowMapper`作为实现类。 通过将相应的class类作为参数传入到`newInstance`方法中，我们可以创建这个实现类。

```java
public class JdbcActorDao implements ActorDao {

    private SimpleJdbcCall procReadAllActors;

    public void setDataSource(DataSource dataSource) {
        JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
        jdbcTemplate.setResultsMapCaseInsensitive(true);
        this.procReadAllActors = new SimpleJdbcCall(jdbcTemplate)
                .withProcedureName("read_all_actors")
                .returningResultSet("actors",
                BeanPropertyRowMapper.newInstance(Actor.class));
    }

    public List getActorsList() {
        Map m = procReadAllActors.execute(new HashMap<String, Object>(0));
        return (List) m.get("actors");
    }

    // ... additional methods

}
```

execute调用传入一个空Map，因为这里不需要传入任何参数。从结果Map中提取Actors列表，并且返回给调用者。

