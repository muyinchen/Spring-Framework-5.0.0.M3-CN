### 15.8.5Testing data access logic with an embedded database

Embedded databases provide a lightweight way to test data access code. The following is a data access integration test template that uses an embedded database. Using a template like this can be useful for_one-offs_when the embedded database does not need to be reused across test classes. However, if you wish to create an embedded database that is shared within a test suite, consider using the[Spring TestContext Framework](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/integration-testing.html#testcontext-framework)and configuring the embedded database as a bean in the Spring`ApplicationContext`as described in[Section15.8.2, “Creating an embedded database using Spring XML”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/jdbc.html#jdbc-embedded-database-xml)and[Section15.8.3, “Creating an embedded database programmatically”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/jdbc.html#jdbc-embedded-database-java).

```java
public class DataAccessIntegrationTestTemplate {

	private EmbeddedDatabase db;

	@Before
	public void setUp() {
		// creates an HSQL in-memory database populated from default scripts
		// classpath:schema.sql and classpath:data.sql
		db = new EmbeddedDatabaseBuilder()
				.generateUniqueName(true)
				.addDefaultScripts()
				.build();
	}

	@Test
	public void testDataAccess() {
		JdbcTemplate template = new JdbcTemplate(db);
		template.query( /* ... */ );
	}

	@After
	public void tearDown() {
		db.shutdown();
	}

}
```



