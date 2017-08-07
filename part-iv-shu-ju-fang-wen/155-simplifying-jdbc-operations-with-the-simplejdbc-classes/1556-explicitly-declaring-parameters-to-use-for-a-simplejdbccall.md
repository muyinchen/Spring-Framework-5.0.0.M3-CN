### 15.5.6**为SimpleJdbcCall显式定义参数**

你已经了解如何通过元数据来简化参数配置，但如果你需要的话也可以显式指定参数。这样做的方法是在创建`SimpleJdbcCall`类同时通过`declareParameters`方法进行配置，这个方式可以传入一系列的`SqlParameter`。下面的章节会详细描述如何定义一个`SqlParameter`

| ![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| :--- |
| 备注：如果你使用的数据库不是Spring支持的数据库类型的话显式定义就很有必要了。当前Spring支持以下数据库的存储过程元数据查找能力：Apache Derby, DB2, MySQL, Microsoft SQL Server, Oracle, 和 Sybase. 我们同时对某些数据库内置函数支持元数据特性：比如：MySQL、Microsoft SQL Server和Oracle。 |

你可以选择显式定义一个、多个，或者所有参数。当你没有显式定义参数时元数据参数仍然会被使用。当你不想用元数据查找参数功能、只想指定参数时，需要调用`withoutProcedureColumnMetaDataAccess`方法。假设你针对同一个数据函数定义了两个或多个不同的调用方法签名，在每一个给定的签名中你需要使用`useInParameterNames`来指定传入参数的名称列表。

下面是一个完全自定义的存储过程调用例子

```java
public class JdbcActorDao implements ActorDao {

    private SimpleJdbcCall procReadActor;

    public void setDataSource(DataSource dataSource) {
        JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
        jdbcTemplate.setResultsMapCaseInsensitive(true);
        this.procReadActor = new SimpleJdbcCall(jdbcTemplate)
                .withProcedureName("read_actor")
                .withoutProcedureColumnMetaDataAccess()
                .useInParameterNames("in_id")
                .declareParameters(
                        new SqlParameter("in_id", Types.NUMERIC),
                        new SqlOutParameter("out_first_name", Types.VARCHAR),
                        new SqlOutParameter("out_last_name", Types.VARCHAR),
                        new SqlOutParameter("out_birth_date", Types.DATE)
                );
    }

    // ... additional methods
}
```

两个例子的执行结果是一样的，区别是这个例子显式指定了所有细节，而不是仅仅依赖于数据库元数据。

