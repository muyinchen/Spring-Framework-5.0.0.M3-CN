### 15.2.2NamedParameterJdbcTemplate

The`NamedParameterJdbcTemplate`class adds support for programming JDBC statements using named parameters, as opposed to programming JDBC statements using only classic placeholder \(`'?'`\) arguments. The`NamedParameterJdbcTemplate`class wraps a`JdbcTemplate`, and delegates to the wrapped`JdbcTemplate`to do much of its work. This section describes only those areas of the`NamedParameterJdbcTemplate`class that differ from the`JdbcTemplate`itself; namely, programming JDBC statements using named parameters.

```java
// some JDBC-backed DAO class...
private NamedParameterJdbcTemplate namedParameterJdbcTemplate;

public void setDataSource(DataSource dataSource) {
	this.namedParameterJdbcTemplate = new NamedParameterJdbcTemplate(dataSource);
}

public int countOfActorsByFirstName(String firstName) {

	String sql = "select count(*) from T_ACTOR where first_name = :first_name";

	SqlParameterSource namedParameters = new MapSqlParameterSource("first_name", firstName);

	return this.namedParameterJdbcTemplate.queryForObject(sql, namedParameters, Integer.class);
}
```

Notice the use of the named parameter notation in the value assigned to the`sql`variable, and the corresponding value that is plugged into the`namedParameters`variable \(of type`MapSqlParameterSource`\).

Alternatively, you can pass along named parameters and their corresponding values to a`NamedParameterJdbcTemplate`instance by using the`Map`-based style.The remaining methods exposed by the`NamedParameterJdbcOperations`and implemented by the`NamedParameterJdbcTemplate`class follow a similar pattern and are not covered here.

The following example shows the use of the`Map`-based style.

```java
// some JDBC-backed DAO class...
private NamedParameterJdbcTemplate namedParameterJdbcTemplate;

public void setDataSource(DataSource dataSource) {
	this.namedParameterJdbcTemplate = new NamedParameterJdbcTemplate(dataSource);
}

public int countOfActorsByFirstName(String firstName) {

	String sql = "select count(*) from T_ACTOR where first_name = :first_name";

	Map<String, String> namedParameters = Collections.singletonMap("first_name", firstName);

	return this.namedParameterJdbcTemplate.queryForObject(sql, namedParameters,  Integer.class);
}
```

One nice feature related to the`NamedParameterJdbcTemplate`\(and existing in the same Java package\) is the`SqlParameterSource`interface. You have already seen an example of an implementation of this interface in one of the previous code snippet \(the`MapSqlParameterSource`class\). An`SqlParameterSource`is a source of named parameter values to a`NamedParameterJdbcTemplate`. The`MapSqlParameterSource`class is a very simple implementation that is simply an adapter around a`java.util.Map`, where the keys are the parameter names and the values are the parameter values.

Another`SqlParameterSource`implementation is the`BeanPropertySqlParameterSource`class. This class wraps an arbitrary JavaBean \(that is, an instance of a class that adheres to[the JavaBean conventions](http://www.oracle.com/technetwork/java/javase/documentation/spec-136004.html)\), and uses the properties of the wrapped JavaBean as the source of named parameter values.

```java
public class Actor {

	private Long id;
	private String firstName;
	private String lastName;

	public String getFirstName() {
		return this.firstName;
	}

	public String getLastName() {
		return this.lastName;
	}

	public Long getId() {
		return this.id;
	}

	// setters omitted...

}
```

```java
// some JDBC-backed DAO class...
private NamedParameterJdbcTemplate namedParameterJdbcTemplate;

public void setDataSource(DataSource dataSource) {
	this.namedParameterJdbcTemplate = new NamedParameterJdbcTemplate(dataSource);
}

public int countOfActors(Actor exampleActor) {

	// notice how the named parameters match the properties of the above 'Actor' class
	String sql = "select count(*) from T_ACTOR where first_name = :firstName and last_name = :lastName";

	SqlParameterSource namedParameters = new BeanPropertySqlParameterSource(exampleActor);

	return this.namedParameterJdbcTemplate.queryForObject(sql, namedParameters, Integer.class);
}
```

Remember that the`NamedParameterJdbcTemplate`class_wraps_a classic`JdbcTemplate`template; if you need access to the wrapped`JdbcTemplate`instance to access functionality only present in the`JdbcTemplate`class, you can use the`getJdbcOperations()`method to access the wrapped`JdbcTemplate`through the`JdbcOperations`interface.

See also[the section called “JdbcTemplate best practices”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/jdbc.html#jdbc-JdbcTemplate-idioms)for guidelines on using the`NamedParameterJdbcTemplate`class in the context of an application.

