### 15.6.2MappingSqlQuery

`MappingSqlQuery`is a reusable query in which concrete subclasses must implement the abstract`mapRow(..)`method to convert each row of the supplied`ResultSet`into an object of the type specified. The following example shows a custom query that maps the data from the`t_actor`relation to an instance of the`Actor`class.

```
public class ActorMappingQuery extends MappingSqlQuery<Actor> {

	public ActorMappingQuery(DataSource ds) {
		super(ds, "select id, first_name, last_name from t_actor where id = ?");
		super.declareParameter(new SqlParameter("id", Types.INTEGER));
		compile();
	}

	@Override
	protected Actor mapRow(ResultSet rs, int rowNumber) throws SQLException {
		Actor actor = new Actor();
		actor.setId(rs.getLong("id"));
		actor.setFirstName(rs.getString("first_name"));
		actor.setLastName(rs.getString("last_name"));
		return actor;
	}

}
```

The class extends`MappingSqlQuery`parameterized with the`Actor`type. The constructor for this customer query takes the`DataSource`as the only parameter. In this constructor you call the constructor on the superclass with the`DataSource`and the SQL that should be executed to retrieve the rows for this query. This SQL will be used to create a`PreparedStatement`so it may contain place holders for any parameters to be passed in during execution.You must declare each parameter using the`declareParameter`method passing in an`SqlParameter`. The`SqlParameter`takes a name and the JDBC type as defined in`java.sql.Types`. After you define all parameters, you call the`compile()`method so the statement can be prepared and later executed. This class is thread-safe after it is compiled, so as long as these instances are created when the DAO is initialized they can be kept as instance variables and be reused.

```java
private ActorMappingQuery actorMappingQuery;

@Autowired
public void setDataSource(DataSource dataSource) {
	this.actorMappingQuery = new ActorMappingQuery(dataSource);
}

public Customer getCustomer(Long id) {
	return actorMappingQuery.findObject(id);
}
```

The method in this example retrieves the customer with the id that is passed in as the only parameter. Since we only want one object returned we simply call the convenience method`findObject`with the id as parameter. If we had instead a query that returned a list of objects and took additional parameters then we would use one of the execute methods that takes an array of parameter values passed in as varargs.

```java
public List<Actor> searchForActors(int age, String namePattern) {
	List<Actor> actors = actorSearchMappingQuery.execute(age, namePattern);
	return actors;
}
```



