### 15.2.4Executing statements

Executing an SQL statement requires very little code. You need a`DataSource`and a`JdbcTemplate`, including the convenience methods that are provided with the`JdbcTemplate`. The following example shows what you need to include for a minimal but fully functional class that creates a new table:

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



