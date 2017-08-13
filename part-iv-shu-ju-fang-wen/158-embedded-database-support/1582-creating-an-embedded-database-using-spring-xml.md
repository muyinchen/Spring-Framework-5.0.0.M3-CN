### 15.8.2**使用Spring配置来创建内嵌数据库**

如果你想要将内嵌的数据库实例作为Bean配置到Spring的`ApplicationContext`中，使用`spring-jdbc`命名空间下的`embedded-database` tag

```java
<jdbc:embedded-database id="dataSource" generate-name="true">
    <jdbc:script location="classpath:schema.sql"/>
    <jdbc:script location="classpath:test-data.sql"/>
</jdbc:embedded-database>
```

上面的配置创建了一个内嵌的HSQL数据库，并且在classpath下面配置`schema.sql`和`test-data.sql`资源。同时，作为一种最佳实践，内嵌数据库会被制定一个唯一生成的名字。内嵌数据库在Spring容器中作为`javax.sql.DataSource` Bean类型存在，并且能够被注入到所需的数据库访问对象中。

