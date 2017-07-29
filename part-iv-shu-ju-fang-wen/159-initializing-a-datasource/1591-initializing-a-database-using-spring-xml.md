### 15.9.1Initializing a database using Spring XML

If you want to initialize a database and you can provide a reference to a`DataSource`bean, use the`initialize-database`tag in the`spring-jdbc`namespace:

```java
<jdbc:initialize-database data-source="dataSource">
	<jdbc:script location="classpath:com/foo/sql/db-schema.sql"/>
	<jdbc:script location="classpath:com/foo/sql/db-test-data.sql"/>
</jdbc:initialize-database>
```

The example above executes the two scripts specified against the database: the first script creates a schema, and the second populates tables with a test data set. The script locations can also be patterns with wildcards in the usual ant style used for resources in Spring \(e.g.`classpath*:/com/foo/**/sql/*-data.sql`\). If a pattern is used, the scripts are executed in lexical order of their URL or filename.

The default behavior of the database initializer is to unconditionally execute the scripts provided. This will not always be what you want, for instance, if you are executing the scripts against a database that already has test data in it. The likelihood of accidentally deleting data is reduced by following the common pattern \(as shown above\) of creating the tables first and then inserting the data — the first step will fail if the tables already exist.

However, to gain more control over the creation and deletion of existing data, the XML namespace provides a few additional options. The first is a flag to switch the initialization on and off. This can be set according to the environment \(e.g. to pull a boolean value from system properties or an environment bean\), for example:

```java
<jdbc:initialize-database data-source="dataSource"
	enabled="#{systemProperties.INITIALIZE_DATABASE}">
	<jdbc:script location="..."/>
</jdbc:initialize-database>
```

The second option to control what happens with existing data is to be more tolerant of failures. To this end you can control the ability of the initializer to ignore certain errors in the SQL it executes from the scripts, for example:

```java
<jdbc:initialize-database data-source="dataSource" ignore-failures="DROPS">
	<jdbc:script location="..."/>
</jdbc:initialize-database>
```

In this example we are saying we expect that sometimes the scripts will be executed against an empty database, and there are some`DROP`statements in the scripts which would therefore fail. So failed SQL`DROP`statements will be ignored, but other failures will cause an exception. This is useful if your SQL dialect doesn’t support`DROP …​ IF EXISTS`\(or similar\) but you want to unconditionally remove all test data before re-creating it. In that case the first script is usually a set of`DROP`statements, followed by a set of`CREATE`statements.

The`ignore-failures`option can be set to`NONE`\(the default\),`DROPS`\(ignore failed drops\), or`ALL`\(ignore all failures\).

Each statement should be separated by`;`or a new line if the`;`character is not present at all in the script. You can control that globally or script by script, for example:

```java
<jdbc:initialize-database data-source="dataSource" separator="@@">
	<jdbc:script location="classpath:com/foo/sql/db-schema.sql" separator=";"/>
	<jdbc:script location="classpath:com/foo/sql/db-test-data-1.sql"/>
	<jdbc:script location="classpath:com/foo/sql/db-test-data-2.sql"/>
</jdbc:initialize-database>
```

In this example, the two`test-data`scripts use`@@`as statement separator and only the`db-schema.sql`uses`;`. This configuration specifies that the default separator is`@@`and override that default for the`db-schema`script.

If you need more control than you get from the XML namespace, you can simply use the`DataSourceInitializer`directly and define it as a component in your application.

#### Initialization of other components that depend on the database

A large class of applications can just use the database initializer with no further complications: those that do not use the database until after the Spring context has started. If your application is_not_one of those then you might need to read the rest of this section.

The database initializer depends on a`DataSource`instance and executes the scripts provided in its initialization callback \(analogous to an`init-method`in an XML bean definition, a`@PostConstruct`method in a component, or the`afterPropertiesSet()`method in a component that implements`InitializingBean`\). If other beans depend on the same data source and also use the data source in an initialization callback, then there might be a problem because the data has not yet been initialized. A common example of this is a cache that initializes eagerly and loads data from the database on application startup.

To get around this issue you have two options: change your cache initialization strategy to a later phase, or ensure that the database initializer is initialized first.

The first option might be easy if the application is in your control, and not otherwise. Some suggestions for how to implement this include:

* Make the cache initialize lazily on first usage, which improves application startup time.

* Have your cache or a separate component that initializes the cache implement`Lifecycle`or`SmartLifecycle`. When the application context starts up a`SmartLifecycle`can be automatically started if its`autoStartup`flag is set, and a`Lifecycle`can be started manually by calling`ConfigurableApplicationContext.start()`on the enclosing context.

* Use a Spring`ApplicationEvent`or similar custom observer mechanism to trigger the cache initialization.`ContextRefreshedEvent`is always published by the context when it is ready for use \(after all beans have been initialized\), so that is often a useful hook \(this is how the`SmartLifecycle`works by default\).

The second option can also be easy. Some suggestions on how to implement this include:

* Rely on the default behavior of the Spring`BeanFactory`, which is that beans are initialized in registration order. You can easily arrange that by adopting the common practice of a set of\`\`elements in XML configuration that order your application modules, and ensure that the database and database initialization are listed first.

* Separate the`DataSource`and the business components that use it, and control their startup order by putting them in separate`ApplicationContext`instances \(e.g. the parent context contains the`DataSource`, and child context contains the business components\). This structure is common in Spring web applications but can be more generally applied.



