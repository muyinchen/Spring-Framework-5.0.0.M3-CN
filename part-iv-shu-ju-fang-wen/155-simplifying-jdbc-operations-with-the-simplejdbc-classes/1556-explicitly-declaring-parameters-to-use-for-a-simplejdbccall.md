### 15.5.6Explicitly declaring parameters to use for a SimpleJdbcCall

You have seen how the parameters are deduced based on metadata, but you can declare then explicitly if you wish. You do this by creating and configuring`SimpleJdbcCall`with the`declareParameters`method, which takes a variable number of`SqlParameter`objects as input. See the next section for details on how to define an`SqlParameter`.

| ![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| :--- |
| Explicit declarations are necessary if the database you use is not a Spring-supported database. Currently Spring supports metadata lookup of stored procedure calls for the following databases: Apache Derby, DB2, MySQL, Microsoft SQL Server, Oracle, and Sybase. We also support metadata lookup of stored functions for MySQL, Microsoft SQL Server, and Oracle. |

You can opt to declare one, some, or all the parameters explicitly. The parameter metadata is still used where you do not declare parameters explicitly. To bypass all processing of metadata lookups for potential parameters and only use the declared parameters, you call the method`withoutProcedureColumnMetaDataAccess`as part of the declaration. Suppose that you have two or more different call signatures declared for a database function. In this case you call the`useInParameterNames`to specify the list of IN parameter names to include for a given signature.

The following example shows a fully declared procedure call, using the information from the preceding example.

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

The execution and end results of the two examples are the same; this one specifies all details explicitly rather than relying on metadata.

