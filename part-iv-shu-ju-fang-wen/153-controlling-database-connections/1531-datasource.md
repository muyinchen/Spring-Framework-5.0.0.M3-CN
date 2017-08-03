### 15.3.1DataSource

Spring用`DataSource`来保持与数据库的连接。`DataSource`是JDBC规范的一部分同时是一种通用的连接工厂。它使得框架或者容器对应用代码屏蔽连接池或者事务管理等底层逻辑。作为开发者，你无需知道连接数据库的底层逻辑；这只是创建datasource的管理员该负责的模块。在开发测试过程中你可能需要同时扮演双重角色，但最终上线时你不需要知道生产数据源是如何配置的。

当使用Spring JDBC时，你可以通过JNDI获取数据库数据源、也可以利用第三方依赖包的连接池实现来配置。比较受欢迎的三方库有Apache Jakarta Commons DBCP 和 C3P0。在Spring产品内，有自己的数据源连接实现，但仅仅用于测试目的，同时并没有使用到连接池。

这一节使用了Spring的`DriverManagerDataSource`实现、其他更多的实现会在后面提到。

| ![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| :--- |
| 注意：仅仅使用`DriverManagerDataSource`类只是为了测试目的、因为此类没有连接池功能，因此在并发连接请求时性能会比较差. |

通过`DriverManagerDataSource`获取数据库连接的方式和传统JDBC是类似的。首先指定JDBC驱动的类全名，`DriverManager `会据此来加载驱动类。接下来、提供JDBC驱动对应的URL名称。（可以从相应驱动的文档里找到具体的名称）。然后传入用户名和密码来连接数据库。下面是一个具体配置`DriverManagerDataSource`连接的Java代码块:

```java
DriverManagerDataSource dataSource = new DriverManagerDataSource();
dataSource.setDriverClassName("org.hsqldb.jdbcDriver");
dataSource.setUrl("jdbc:hsqldb:hsql://localhost:");
dataSource.setUsername("sa");
dataSource.setPassword("");
```

接下来是相关的XML配置：

```java
<bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
    <property name="driverClassName" value="${jdbc.driverClassName}"/>
    <property name="url" value="${jdbc.url}"/>
    <property name="username" value="${jdbc.username}"/>
    <property name="password" value="${jdbc.password}"/>
</bean>

<context:property-placeholder location="jdbc.properties"/>
```

下面的例子展示的是DBCP和C3P0的基础连接配置。如果需要连接更多的连接池选项、请查看各自连接池实现的具体产品文档

DBCP配置：

```java
<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
    <property name="driverClassName" value="${jdbc.driverClassName}"/>
    <property name="url" value="${jdbc.url}"/>
    <property name="username" value="${jdbc.username}"/>
    <property name="password" value="${jdbc.password}"/>
</bean>

<context:property-placeholder location="jdbc.properties"/>
```

C3P0 配置：

```java
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource" destroy-method="close">
    <property name="driverClass" value="${jdbc.driverClassName}"/>
    <property name="jdbcUrl" value="${jdbc.url}"/>
    <property name="user" value="${jdbc.username}"/>
    <property name="password" value="${jdbc.password}"/>
</bean>

<context:property-placeholder location="jdbc.properties"/>
```



