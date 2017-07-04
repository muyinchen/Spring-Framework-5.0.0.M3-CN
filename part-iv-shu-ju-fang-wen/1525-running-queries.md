### 15.2.5Running queries

Some query methods return a single value. To retrieve a count or a specific value from one row, use`queryForObject(..)`. The latter converts the returned JDBC`Type`to the Java class that is passed in as an argument. If the type conversion is invalid, then an`InvalidDataAccessApiUsageException`is thrown. Here is an example that contains two query methods, one for an`int`and one that queries for a`String`.

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

In addition to the single result query methods, several methods return a list with an entry for each row that the query returned. The most generic method is`queryForList(..)`which returns a`List`where each entry is a`Map`with each entry in the map representing the column value for that row. If you add a method to the above example to retrieve a list of all the rows, it would look like this:

```java
private JdbcTemplate jdbcTemplate;

public void setDataSource(DataSource dataSource) {
	this.jdbcTemplate = new JdbcTemplate(dataSource);
}

public List<Map<String, Object>> getList() {
	return this.jdbcTemplate.queryForList("select * from mytable");
}
```

The list returned would look something like this:

```java
[{name=Bob, id=1}, {name=Mary, id=2}]
```



