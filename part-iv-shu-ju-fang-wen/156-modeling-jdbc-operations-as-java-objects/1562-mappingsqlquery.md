### 15.6.2MappingSqlQuery

`MappingSqlQuery`是一个可重用的查询类，它的子类必须实现`mapRow(..)`方法，将结果集返回的每一行转换成指定的对象类型。下面的例子展示了一个自定义的查询例子，将`t_actor`关系表的数据映射成`Actor`类。

```java
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

这个类继承了`MappingSqlQuery`，并且传入`Actor`类型的泛型参数。这个自定义查询类的构造函数将`DataSource`作为唯一的传入参数。这个构造器中你调用父类的构造器，传入`DataSource`以及相应的SQL参数。该SQL用于创建`PreparedStatement`，因此它可能包含任何在执行过程中传入参数的占位符。你必须在`SqlParameter`中使用`declareParameter`方法定义每个参数。`SqlParameter`使用`java.sql.Types`定义名字和JDBC类型。在你定义了所有的参数后，你需要调用`compile()`方法，语句被预编译后方便后续的执行。这个类在编译后是线程安全的，一旦在DAO初始化时这些实例被创建后，它们可以作为实例变量一直被重用。

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

这个例子中的方法通过唯一的传入参数id获取customer实例。因为我们只需要返回一个对象，所以就简单的调用`findObject`类就可以了，这个方法只需要传入id参数。如果我们需要一次查询返回一个列表的话，就需要使用传入可变参数数组的执行方法。

```java
public List<Actor> searchForActors(int age, String namePattern) {
    List<Actor> actors = actorSearchMappingQuery.execute(age, namePattern);
    return actors;
}
```



