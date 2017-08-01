### 16.4.5Setting up JPA with JTA transaction management

As an alternative to`JpaTransactionManager`, Spring also allows for multi-resource transaction coordination via JTA, either in a Java EE environment or with a standalone transaction coordinator such as Atomikos. Aside from choosing Spring’s`JtaTransactionManager`instead of`JpaTransactionManager`, there are a few further steps to take:

* The underlying JDBC connection pools need to be XA-capable and integrated with your transaction coordinator. This is usually straightforward in a Java EE environment, simply exposing a different kind of`DataSource`via JNDI. Check your application server documentation for details. Analogously, a standalone transaction coordinator usually comes with special XA-integrated`DataSource`implementations; again, check its docs.

* The JPA`EntityManagerFactory`setup needs to be configured for JTA. This is provider-specific, typically via special properties to be specified as "jpaProperties" on`LocalContainerEntityManagerFactoryBean`. In the case of Hibernate, these properties are even version-specific; please check your Hibernate documentation for details.

* Spring’s`HibernateJpaVendorAdapter`enforces certain Spring-oriented defaults such as the connection release mode "on-close" which matches Hibernate’s own default in Hibernate 5.0 but not anymore in 5.1/5.2. For a JTA setup, either do not declare`HibernateJpaVendorAdapter`to begin with, or turn off its`prepareConnection`flag. Alternatively, set Hibernate 5.2’s "hibernate.connection.handling\_mode" property to "DELAYED\_ACQUISITION\_AND\_RELEASE\_AFTER\_STATEMENT" to restore Hibernate’s own default. See[Section16.3.7, “Spurious application server warnings with Hibernate”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/orm.html#orm-hibernate-invalid-jdbc-access-error)for a related note about WebLogic.

* Alternatively, consider obtaining the`EntityManagerFactory`from your application server itself, i.e. via a JNDI lookup instead of a locally declared`LocalContainerEntityManagerFactoryBean`. A server-provided`EntityManagerFactory`might require special definitions in your server configuration, making the deployment less portable, but will be set up for the server’s JTA environment out of the box.



