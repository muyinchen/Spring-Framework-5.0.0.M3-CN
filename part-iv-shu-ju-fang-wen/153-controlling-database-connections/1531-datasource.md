### 15.3.1DataSource

Spring obtains a connection to the database through a`DataSource`. A`DataSource`is part of the JDBC specification and is a generalized connection factory. It allows a container or a framework to hide connection pooling and transaction management issues from the application code. As a developer, you need not know details about how to connect to the database; that is the responsibility of the administrator that sets up the datasource. You most likely fill both roles as you develop and test code, but you do not necessarily have to know how the production data source is configured.

When using Spring’s JDBC layer, you obtain a data source from JNDI or you configure your own with a connection pool implementation provided by a third party. Popular implementations are Apache Jakarta Commons DBCP and C3P0. Implementations in the Spring distribution are meant only for testing purposes and do not provide pooling.

This section uses Spring’s`DriverManagerDataSource`implementation, and several additional implementations are covered later.

| ![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| :--- |
| Only use the`DriverManagerDataSource`class should only be used for testing purposes since it does not provide pooling and will perform poorly when multiple requests for a connection are made. |

You obtain a connection with`DriverManagerDataSource`as you typically obtain a JDBC connection. Specify the fully qualified classname of the JDBC driver so that the`DriverManager`can load the driver class. Next, provide a URL that varies between JDBC drivers. \(Consult the documentation for your driver for the correct value.\) Then provide a username and a password to connect to the database. Here is an example of how to configure a`DriverManagerDataSource`in Java code:

```java
DriverManagerDataSource dataSource = new DriverManagerDataSource();
dataSource.setDriverClassName("org.hsqldb.jdbcDriver");
dataSource.setUrl("jdbc:hsqldb:hsql://localhost:");
dataSource.setUsername("sa");
dataSource.setPassword("");
```

Here is the corresponding XML configuration:

```java
<bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
	<property name="driverClassName" value="${jdbc.driverClassName}"/>
	<property name="url" value="${jdbc.url}"/>
	<property name="username" value="${jdbc.username}"/>
	<property name="password" value="${jdbc.password}"/>
</bean>

<context:property-placeholder location="jdbc.properties"/>
```

The following examples show the basic connectivity and configuration for DBCP and C3P0. To learn about more options that help control the pooling features, see the product documentation for the respective connection pooling implementations.

DBCP configuration:

```java
<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
	<property name="driverClassName" value="${jdbc.driverClassName}"/>
	<property name="url" value="${jdbc.url}"/>
	<property name="username" value="${jdbc.username}"/>
	<property name="password" value="${jdbc.password}"/>
</bean>

<context:property-placeholder location="jdbc.properties"/>
```

C3P0 configuration:

```java
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource" destroy-method="close">
	<property name="driverClass" value="${jdbc.driverClassName}"/>
	<property name="jdbcUrl" value="${jdbc.url}"/>
	<property name="user" value="${jdbc.username}"/>
	<property name="password" value="${jdbc.password}"/>
</bean>

<context:property-placeholder location="jdbc.properties"/>
```



