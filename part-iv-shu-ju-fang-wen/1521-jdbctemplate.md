### 15.2.1JdbcTemplate

The`JdbcTemplate`class is the central class in the JDBC core package. It handles the creation and release of resources, which helps you avoid common errors such as forgetting to close the connection. It performs the basic tasks of the core JDBC workflow such as statement creation and execution, leaving application code to provide SQL and extract results. The`JdbcTemplate`class executes SQL queries, update statements and stored procedure calls, performs iteration over`ResultSet`s and extraction of returned parameter values. It also catches JDBC exceptions and translates them to the generic, more informative, exception hierarchy defined in the`org.springframework.dao`package.

When you use the`JdbcTemplate`for your code, you only need to implement callback interfaces, giving them a clearly defined contract. The`PreparedStatementCreator`callback interface creates a prepared statement given a`Connection`provided by this class, providing SQL and any necessary parameters. The same is true for the`CallableStatementCreator`interface, which creates callable statements. The`RowCallbackHandler`interface extracts values from each row of a`ResultSet`.

The`JdbcTemplate`can be used within a DAO implementation through direct instantiation with a`DataSource`reference, or be configured in a Spring IoC container and given to DAOs as a bean reference.

| ![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| :--- |
| The`DataSource`should always be configured as a bean in the Spring IoC container. In the first case the bean is given to the service directly; in the second case it is given to the prepared template. |

All SQL issued by this class is logged at the`DEBUG`level under the category corresponding to the fully qualified class name of the template instance \(typically`JdbcTemplate`, but it may be different if you are using a custom subclass of the`JdbcTemplate`class\).

#### Examples of JdbcTemplate class usage

This section provides some examples of`JdbcTemplate`class usage. These examples are not an exhaustive list of all of the functionality exposed by the`JdbcTemplate`; see the attendant javadocs for that.

##### Querying \(SELECT\)

Here is a simple query for getting the number of rows in a relation:

```java
int rowCount = this.jdbcTemplate.queryForObject("select count(*) from t_actor", Integer.class);
```

A simple query using a bind variable:

```java
int countOfActorsNamedJoe = this.jdbcTemplate.queryForObject(
		"select count(*) from t_actor where first_name = ?", Integer.class, "Joe");
```

Querying for a`String`:

```java
String lastName = this.jdbcTemplate.queryForObject(
		"select last_name from t_actor where id = ?",
		new Object[]{1212L}, String.class);
```

Querying and populating a _single _domain object:

```java
Actor actor = this.jdbcTemplate.queryForObject(
		"select first_name, last_name from t_actor where id = ?",
		new Object[]{1212L},
		new RowMapper<Actor>() {
			public Actor mapRow(ResultSet rs, int rowNum) throws SQLException {
				Actor actor = new Actor();
				actor.setFirstName(rs.getString("first_name"));
				actor.setLastName(rs.getString("last_name"));
				return actor;
			}
		});
```

Querying and populating a number of domain objects:

```java
List<Actor> actors = this.jdbcTemplate.query(
		"select first_name, last_name from t_actor",
		new RowMapper<Actor>() {
			public Actor mapRow(ResultSet rs, int rowNum) throws SQLException {
				Actor actor = new Actor();
				actor.setFirstName(rs.getString("first_name"));
				actor.setLastName(rs.getString("last_name"));
				return actor;
			}
		});
```

If the last two snippets of code actually existed in the same application, it would make sense to remove the duplication present in the two`RowMapper`anonymous inner classes, and extract them out into a single class \(typically a`static`nested class\) that can then be referenced by DAO methods as needed. For example, it may be better to write the last code snippet as follows:

```java
public List<Actor> findAllActors() {
	return this.jdbcTemplate.query( "select first_name, last_name from t_actor", new ActorMapper());
}

private static final class ActorMapper implements RowMapper<Actor> {

	public Actor mapRow(ResultSet rs, int rowNum) throws SQLException {
		Actor actor = new Actor();
		actor.setFirstName(rs.getString("first_name"));
		actor.setLastName(rs.getString("last_name"));
		return actor;
	}
}
```

##### Updating \(INSERT/UPDATE/DELETE\) with jdbcTemplate

You use the`update(..)`method to perform insert, update and delete operations. Parameter values are usually provided as var args or alternatively as an object array.

```java
this.jdbcTemplate.update(
		"insert into t_actor (first_name, last_name) values (?, ?)",
		"Leonor", "Watling");
```

```java
this.jdbcTemplate.update(
		"update t_actor set last_name = ? where id = ?",
		"Banjo", 5276L);
```

```java
this.jdbcTemplate.update(
		"delete from actor where id = ?",
		Long.valueOf(actorId));
```

##### Other jdbcTemplate operations

You can use the`execute(..)`method to execute any arbitrary SQL, and as such the method is often used for DDL statements. It is heavily overloaded with variants taking callback interfaces, binding variable arrays, and so on.

```java
this.jdbcTemplate.execute("create table mytable (id integer, name varchar(100))");
```

The following example invokes a simple stored procedure. More sophisticated stored procedure support is[covered later](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/jdbc.html#jdbc-StoredProcedure).

```java
this.jdbcTemplate.update(
		"call SUPPORT.REFRESH_ACTORS_SUMMARY(?)",
		Long.valueOf(unionId));
```

#### JdbcTemplate best practices

Instances of the`JdbcTemplate`class are_threadsafe once configured_. This is important because it means that you can configure a single instance of a`JdbcTemplate`and then safely inject this_shared_reference into multiple DAOs \(or repositories\). The`JdbcTemplate`is stateful, in that it maintains a reference to a`DataSource`, but this state is_not_conversational state.

A common practice when using the`JdbcTemplate`class \(and the associated[`NamedParameterJdbcTemplate`](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/jdbc.html#jdbc-NamedParameterJdbcTemplate)classes\) is to configure a`DataSource`in your Spring configuration file, and then dependency-inject that shared`DataSource`bean into your DAO classes; the`JdbcTemplate`is created in the setter for the`DataSource`. This leads to DAOs that look in part like the following:

```java
public class JdbcCorporateEventDao implements CorporateEventDao {

	private JdbcTemplate jdbcTemplate;

	public void setDataSource(DataSource dataSource) {
		this.jdbcTemplate = new JdbcTemplate(dataSource);
	}

	// JDBC-backed implementations of the methods on the CorporateEventDao follow...
}
```

The corresponding configuration might look like this.

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="
		http://www.springframework.org/schema/beans
		http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context
		http://www.springframework.org/schema/context/spring-context.xsd">

	<bean id="corporateEventDao" class="com.example.JdbcCorporateEventDao">
		<property name="dataSource" ref="dataSource"/>
	</bean>

	<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
		<property name="driverClassName" value="${jdbc.driverClassName}"/>
		<property name="url" value="${jdbc.url}"/>
		<property name="username" value="${jdbc.username}"/>
		<property name="password" value="${jdbc.password}"/>
	</bean>

	<context:property-placeholder location="jdbc.properties"/>

</beans>
```

An alternative to explicit configuration is to use component-scanning and annotation support for dependency injection. In this case you annotate the class with`@Repository`\(which makes it a candidate for component-scanning\) and annotate the`DataSource`setter method with`@Autowired`.

```java
@Repository
public class JdbcCorporateEventDao implements CorporateEventDao {

	private JdbcTemplate jdbcTemplate;

	@Autowired
	public void setDataSource(DataSource dataSource) {
		this.jdbcTemplate = new JdbcTemplate(dataSource);
	}

	// JDBC-backed implementations of the methods on the CorporateEventDao follow...
}
```

The corresponding XML configuration file would look like the following:

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="
		http://www.springframework.org/schema/beans
		http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context
		http://www.springframework.org/schema/context/spring-context.xsd">

	<!-- Scans within the base package of the application for @Component classes to configure as beans -->
	<context:component-scan base-package="org.springframework.docs.test" />

	<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
		<property name="driverClassName" value="${jdbc.driverClassName}"/>
		<property name="url" value="${jdbc.url}"/>
		<property name="username" value="${jdbc.username}"/>
		<property name="password" value="${jdbc.password}"/>
	</bean>

	<context:property-placeholder location="jdbc.properties"/>

</beans>
```

If you are using Spring’s`JdbcDaoSupport`class, and your various JDBC-backed DAO classes extend from it, then your sub-class inherits a`setDataSource(..)`method from the`JdbcDaoSupport`class. You can choose whether to inherit from this class. The`JdbcDaoSupport`class is provided as a convenience only.

Regardless of which of the above template initialization styles you choose to use \(or not\), it is seldom necessary to create a new instance of a`JdbcTemplate`class each time you want to execute SQL. Once configured, a`JdbcTemplate`instance is threadsafe. You may want multiple`JdbcTemplate`instances if your application accesses multiple databases, which requires multiple`DataSources`, and subsequently multiple differently configured`JdbcTemplates`.

