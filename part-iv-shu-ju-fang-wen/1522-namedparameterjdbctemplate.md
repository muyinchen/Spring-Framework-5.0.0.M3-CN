### 15.2.2NamedParameterJdbcTemplate

`NamedParameterJdbcTemplate `提供对JDBC语句命名参数的支持，而普通的JDBC语句只能使用经典的 `‘?’`参数。`NamedParameterJdbcTemplate`内部包装了`JdbcTemplate`，很多功能是直接通过`JdbcTemplate`来实现的。本节主要描述`NamedParameterJdbcTemplate`不同于`JdbcTemplate `的点；即通过使用命名参数来操作JDBC

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

上面代码块可以看到SQL变量中命名参数的标记用法，以及`namedParameters`变量的相关赋值（类型为`MapSqlParameterSource`）

除此以外，你还可以在`NamedParameterJdbcTemplate`中传入`Map`风格的命名参数及相关的值。`NamedParameterJdbcTemplate`类从`NamedParameterJdbcOperations`接口实现的其他方法用法是类似的，这里就不一一叙述了。

下面是一个`Map`风格的例子：

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

与`NamedParameterJdbcTemplate`相关联的`SqlParameterSource`接口提供了很有用的功能（两者在同一个包里面）。在上面的代码片段中你已经看到了这个接口的一个实现例子（就是`MapSqlParameterSource`类）。`SqlParameterSource`类是`NamedParameterJdbcTemplate`类的数值值来源。`MapSqlParameterSource`实现非常简单、只是适配了`java.util.Map`，其中keys就是参数名字，value就是参数值。

另外一个`SqlParameterSource` 的实现是`BeanPropertySqlParameterSource`类。这个类封装了任意一个JavaBean（也就是任意符合[JavaBen规范](http://www.oracle.com/technetwork/java/javase/documentation/spec-136004.html)的实例），在这个实现中，使用了JavaBean的属性作为命名参数的来源。

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

之前提到过`NamedParameterJdbcTemplate`本身包装了经典的`JdbcTemplate`模板。如果你想调用只存在于`JdbcTemplate`类中的方法，你可以使用`getJdbcOperations()`方法、该方法返回`JdbcOperations`接口，通过这个接口你可以调用内部`JdbcTemplate`的方法。

`NamedParameterJdbcTemplate `类在应用上下文的使用方式也可见：“[JdbcTemplate最佳实践](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/jdbc.html#jdbc-JdbcTemplate-idioms)”

