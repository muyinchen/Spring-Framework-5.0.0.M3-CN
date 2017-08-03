### 15.2.6**更新数据库**

下面的例子根据主键更新其中一列值。在这个例子中，一条SQL语句包含行参数的占位符。参数值可以通过可变参数或者对象数组传入。元数据类型需要显式或者自动装箱成对应的包装类型

```java
import javax.sql.DataSource;

import org.springframework.jdbc.core.JdbcTemplate;

public class ExecuteAnUpdate {

    private JdbcTemplate jdbcTemplate;

    public void setDataSource(DataSource dataSource) {
        this.jdbcTemplate = new JdbcTemplate(dataSource);
    }

    public void setName(int id, String name) {
        this.jdbcTemplate.update("update mytable set name = ? where id = ?", name, id);
    }
}
```



