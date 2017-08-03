### 15.2.5运行查询

一些查询方法会返回一个单一的结果。使用`queryForObject(..)`返回结果计数或特定值。当返回特定值类型时，将Java类型作为方法参数传入、最终返回的JDBC类型会被转换成相应的Java类型。如果这个过程中间出现类型转换错误，则会抛出`InvalidDataAccessApiUsageException`的异常。下面的例子包含两个查询方法，一个返回`int`类型、另一个返回了`String`类型。

```java
import javax.sql.DataSource;
import org.springframework.jdbc.core.JdbcTemplate;

public class RunAQuery {

    private JdbcTemplate jdbcTemplate;

    public void setDataSource(DataSource dataSource) {
        this.jdbcTemplate = new JdbcTemplate(dataSource);
    }

    public int getCount() {
        return this.jdbcTemplate.queryForObject("select count(*) from mytable", Integer.class);
    }

    public String getName() {
        return this.jdbcTemplate.queryForObject("select name from mytable", String.class);
    }
}
```

除了返回单一查询结果的方法外，其他方法返回一个列表、列表中每一项代表查询返回的行记录。其中最通用的方式是`queryForList(..)`，返回一个`List`，列表每一项是一个`Map`类型，包含数据库对应行每一列的具体值。下面的代码块给上面的例子添加一个返回所有行的方法：

```java
private JdbcTemplate jdbcTemplate;

public void setDataSource(DataSource dataSource) {
    this.jdbcTemplate = new JdbcTemplate(dataSource);
}

public List<Map<String, Object>> getList() {
    return this.jdbcTemplate.queryForList("select * from mytable");
}
```

返回的列表结果数据格式是这样的：

```java
[{name=Bob, id=1}, {name=Mary, id=2}]
```



