### 15.5.5Calling a stored procedure with SimpleJdbcCall

The`SimpleJdbcCall`class leverages metadata in the database to look up names of`in`and`out`parameters, so that you do not have to declare them explicitly. You can declare parameters if you prefer to do that, or if you have parameters such as`ARRAY`or`STRUCT`that do not have an automatic mapping to a Java class. The first example shows a simple procedure that returns only scalar values in`VARCHAR`and`DATE`format from a MySQL database. The example procedure reads a specified actor entry and returns`first_name`,`last_name`, and`birth_date`columns in the form of`out`parameters.

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

The`in_id`parameter contains the`id`of the actor you are looking up. The`out`parameters return the data read from the table.

The`SimpleJdbcCall`is declared in a similar manner to the`SimpleJdbcInsert`. You should instantiate and configure the class in the initialization method of your data access layer. Compared to the StoredProcedure class, you don’t have to create a subclass and you don’t have to declare parameters that can be looked up in the database metadata. Following is an example of a SimpleJdbcCall configuration using the above stored procedure. The only configuration option, in addition to the`DataSource`, is the name of the stored procedure.

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

The code you write for the execution of the call involves creating an`SqlParameterSource`containing the IN parameter. It’s important to match the name provided for the input value with that of the parameter name declared in the stored procedure. The case does not have to match because you use metadata to determine how database objects should be referred to in a stored procedure. What is specified in the source for the stored procedure is not necessarily the way it is stored in the database. Some databases transform names to all upper case while others use lower case or use the case as specified.

The`execute`method takes the IN parameters and returns a Map containing any`out`parameters keyed by the name as specified in the stored procedure. In this case they are`out_first_name, out_last_name`and`out_birth_date`.

The last part of the`execute`method creates an Actor instance to use to return the data retrieved. Again, it is important to use the names of the`out`parameters as they are declared in the stored procedure. Also, the case in the names of the`out`parameters stored in the results map matches that of the`out`parameter names in the database, which could vary between databases. To make your code more portable you should do a case-insensitive lookup or instruct Spring to use a`LinkedCaseInsensitiveMap`. To do the latter, you create your own`JdbcTemplate`and set the`setResultsMapCaseInsensitive`property to`true`. Then you pass this customized`JdbcTemplate`instance into the constructor of your`SimpleJdbcCall`. Here is an example of this configuration:

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

By taking this action, you avoid conflicts in the case used for the names of your returned`out`parameters.

