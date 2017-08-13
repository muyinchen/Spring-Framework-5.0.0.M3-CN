### 15.8.5**使用内嵌数据库测试数据访问层逻辑**

内嵌数据库提供了数据访问层代码的轻量级测试方案，下面是使用了内嵌数据库的数据访问层集成测试模板。使用这样的模板当内嵌数据库不需要在测试类中被重用时是有用的。不过，当你希望创建可以在test集中共享的内嵌数据库。考虑使用

[Spring TestContext测试框架](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/integration-testing.html#testcontext-framework)，同时在Spring `ApplicationContext`中将内嵌数据库配置成一个Bean，具体参见15.8.2节, “

[使用Spring配置来创建内嵌数据库](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/jdbc.html#jdbc-embedded-database-xml)” 和15.8.3节, “[使用编程方式创建内嵌数据库](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/jdbc.html#jdbc-embedded-database-java)”.

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



