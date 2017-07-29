### 15.8.6Generating unique names for embedded databases

Development teams often encounter errors with embedded databases if their test suite inadvertently attempts to recreate additional instances of the same database. This can happen quite easily if an XML configuration file or`@Configuration`class is responsible for creating an embedded database and the corresponding configuration is then reused across multiple testing scenarios within the same test suite \(i.e., within the same JVM process\) –- for example, integration tests against embedded databases whose`ApplicationContext`configuration only differs with regard to which bean definition profiles are active.

The root cause of such errors is the fact that Spring’s`EmbeddedDatabaseFactory`\(used internally by both the``XML namespace element and the`EmbeddedDatabaseBuilder`for Java Config) will set the name of the embedded database to`"testdb"`if not otherwise specified. For the case of``, the embedded database is typically assigned a name equal to the bean’s`id`\(i.e., often something like`"dataSource"`\). Thus, subsequent attempts to create an embedded database will not result in a new database. Instead, the same JDBC connection URL will be reused, and attempts to create a new embedded database will actually point to an existing embedded database created from the same configuration.

To address this common issue Spring Framework 4.2 provides support for generating_unique_names for embedded databases. To enable the use of generated names, use one of the following options.

* `EmbeddedDatabaseFactory.setGenerateUniqueDatabaseName()`

* `EmbeddedDatabaseBuilder.generateUniqueName()`

* `<jdbc:embedded-database generate-name="true" …​ >`



