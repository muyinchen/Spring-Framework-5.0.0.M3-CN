### 15.8.3Creating an embedded database programmatically

The`EmbeddedDatabaseBuilder`class provides a fluent API for constructing an embedded database programmatically. Use this when you need to create an embedded database in a standalone environment or in a standalone integration test like in the following example.

```java
EmbeddedDatabase db = new EmbeddedDatabaseBuilder()
    .generateUniqueName(true)
    .setType(H2)
    .setScriptEncoding("UTF-8")
    .ignoreFailedDrops(true)
    .addScript("schema.sql")
    .addScripts("user_data.sql", "country_data.sql")
    .build();

// perform actions against the db (EmbeddedDatabase extends javax.sql.DataSource)

db.shutdown()
```

Consult the Javadoc for`EmbeddedDatabaseBuilder`for further details on all supported options.

The`EmbeddedDatabaseBuilder`can also be used to create an embedded database using Java Config like in the following example.

```java
@Configuration
public class DataSourceConfig {

    @Bean
    public DataSource dataSource() {
        return new EmbeddedDatabaseBuilder()
            .generateUniqueName(true)
            .setType(H2)
            .setScriptEncoding("UTF-8")
            .ignoreFailedDrops(true)
            .addScript("schema.sql")
            .addScripts("user_data.sql", "country_data.sql")
            .build();
    }
}
```



