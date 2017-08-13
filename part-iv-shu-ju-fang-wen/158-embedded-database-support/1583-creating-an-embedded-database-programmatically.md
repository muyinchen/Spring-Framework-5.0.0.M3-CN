### 15.8.3**使用编程方式创建内嵌数据库**

`EmbeddedDatabaseBuilder`提供了创建内嵌数据库的流式API。当你在独立的环境中或者是在独立的集成测试中可以使用这种方法创建一个内嵌数据库，下面是一个例子：

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

更多支持的细节请参见：`EmbeddedDatabaseBuilder `的JavaDoc。

`EmbeddedDatabaseBuilder `也可以使用Java Config类来创建内嵌数据库，下面是一个例子：

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



