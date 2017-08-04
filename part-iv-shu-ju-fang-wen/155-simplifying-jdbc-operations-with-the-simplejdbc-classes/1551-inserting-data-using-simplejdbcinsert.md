### 15.5.1**利用SimpleJdbcInsert插入数据**

Let’s start by looking at the`SimpleJdbcInsert`class with the minimal amount of configuration options. You should instantiate the`SimpleJdbcInsert`in the data access layer’s initialization method. For this example, the initializing method is the`setDataSource`method. You do not need to subclass the`SimpleJdbcInsert`class; simply create a new instance and set the table name using the`withTableName`method. Configuration methods for this class follow the "fluid" style that returns the instance of the`SimpleJdbcInsert`, which allows you to chain all configuration methods. This example uses only one configuration method; you will see examples of multiple ones later.

让我们首先看`SimpleJdbcInsert`类可提供的最小配置选项。你需要在数据访问层初始化方法里面初始化`SimpleJdbcInsert`类。在这个例子中，初始化方法是`setDataSource`。你不需要继承`SimpleJdbcInsert`，只需要简单的创建其实例同时调用`withTableName`设置数据库名。该类的配置方法遵循“流体”样式，返回`SimpleJdbcInsert`的实例，它允许您链接所有配置方法。 此示例仅使用一种配置方法; 稍后会看到多个例子。

```java
public class JdbcActorDao implements ActorDao {

    private JdbcTemplate jdbcTemplate;
    private SimpleJdbcInsert insertActor;

    public void setDataSource(DataSource dataSource) {
        this.jdbcTemplate = new JdbcTemplate(dataSource);
        this.insertActor = new SimpleJdbcInsert(dataSource).withTableName("t_actor");
    }

    public void add(Actor actor) {
        Map<String, Object> parameters = new HashMap<String, Object>(3);
        parameters.put("id", actor.getId());
        parameters.put("first_name", actor.getFirstName());
        parameters.put("last_name", actor.getLastName());
        insertActor.execute(parameters);
    }

    // ... additional methods
}
```

代码中的execute只传入`java.utils.Map`作为唯一参数。需要注意的是Map里面用到的Key必须和数据库中表对应的列名一一匹配。这是因为我们需要按顺序读取元数据来构造实际的插入语句。

