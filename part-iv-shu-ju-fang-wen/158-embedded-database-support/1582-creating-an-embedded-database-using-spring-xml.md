### 15.8.2Creating an embedded database using Spring XML

If you want to expose an embedded database instance as a bean in a Spring`ApplicationContext`, use the`embedded-database`tag in the`spring-jdbc`namespace:

```java
<jdbc:embedded-database id="dataSource" generate-name="true">
	<jdbc:script location="classpath:schema.sql"/>
	<jdbc:script location="classpath:test-data.sql"/>
</jdbc:embedded-database>
```

The preceding configuration creates an embedded HSQL database populated with SQL from`schema.sql`and`test-data.sql`resources in the root of the classpath. In addition, as a best practice, the embedded database will be assigned a uniquely generated name. The embedded database is made available to the Spring container as a bean of type`javax.sql.DataSource`which can then be injected into data access objects as needed.

