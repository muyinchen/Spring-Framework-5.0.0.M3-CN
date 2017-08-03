### 15.2.4**执行SQL语句**

执行一条SQL语句非常方便。你只需要依赖`DataSource`和`JdbcTemplate`，包括`JdbcTemplate`提供的工具方法。

下面的例子展示了如何创建一个新的数据表，虽然只有几行代码、但已经完全可用了：

```java
import javax.sql.DataSource;
import org.springframework.jdbc.core.JdbcTemplate;

public class ExecuteAStatement {

    private JdbcTemplate jdbcTemplate;

    public void setDataSource(DataSource dataSource) {
        this.jdbcTemplate = new JdbcTemplate(dataSource);
    }

    public void doExecute() {
        this.jdbcTemplate.execute("create table mytable (id integer, name varchar(100))");
    }
}
```



