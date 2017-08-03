### 15.2.1JdbcTemplate

`JdbcTemplate`是JDBC core包里面的核心类。它封装了对资源的创建和释放，可以帮你避免忘记关闭连接等常见错误。它也包含了核心JDBC工作流的一些基础工作、例如执行和声明语句，而把SQL语句的生成以及查询结果的提取工作留给应用代码。`JdbcTemplate`执行查询、更新SQL语句和调用存储过程，运行`ResultSet`迭代和抽取返回参数值。它也可以捕获JDBC异常并把它们转换成更加通用、解释性更强的异常层次结构、这些异常都定义在`org.springframework.dao`包里面。

当你在代码中使用了`JdbcTemplate`类，你只需要实现回调接口。`PreparedStatementCreator`回调接口通过传入的`Connection`类（该类包含SQL和任何必要的参数）创建已声明的语句。`CallableStatementCreator`也提供类似的方式、该接口用于创建回调语句。`RowCallbackHandler`用于获取结果集每一行的值。

可以在DAO实现类中通过传入`DataSource`引用来完成`JdbcTemplate`的初始化；也可以在Spring IOC容器里面配置、作为DAO bean的依赖Bean配置。

| ![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| :--- |
| 备注：`DataSource`最好在Spring IOC容器里作为Bean配置起来。在上面第一种情况下，`DataSource`bean直接传给相关的服务；第二种情况下DataSource bean传递给JdbcTemplate bean。 |

JdbcTemplate中使用的所有SQL以`DEBUG`级别记入日志（一般情况下日志的归类是`JdbcTemplate`对应的全限定类名，不过如果需要对`JdbcTemplate`进行定制的话，可能是它的子类名）

#### Examples of JdbcTemplate class usage

这一节提供了`JdbcTemplate`类的一些使用例子。这些例子没有囊括`JdbcTemplate`可提供的所有功能；全部功能和用法请详见相关的javadocs.

##### 查询\(SELECT\)

下面是一个简单的例子、用于获取关系表里面的行数

```java
int rowCount = this.jdbcTemplate.queryForObject("select count(*) from t_actor", Integer.class);
```

使用绑定变量的简单查询：

```java
int countOfActorsNamedJoe = this.jdbcTemplate.queryForObject(
        "select count(*) from t_actor where first_name = ?", Integer.class, "Joe");
```

`String`查询：

```java
String lastName = this.jdbcTemplate.queryForObject(
        "select last_name from t_actor where id = ?",
        new Object[]{1212L}, String.class);
```

查询和填充领域模型：

```java
Actor actor = this.jdbcTemplate.queryForObject(
        "select first_name, last_name from t_actor where id = ?",
        new Object[]{1212L},
        new RowMapper<Actor>() {
            public Actor mapRow(ResultSet rs, int rowNum) throws SQLException {
                Actor actor = new Actor();
                actor.setFirstName(rs.getString("first_name"));
                actor.setLastName(rs.getString("last_name"));
                return actor;
            }
        });
```

查询和填充多个领域对象：

```java
List<Actor> actors = this.jdbcTemplate.query(
        "select first_name, last_name from t_actor",
        new RowMapper<Actor>() {
            public Actor mapRow(ResultSet rs, int rowNum) throws SQLException {
                Actor actor = new Actor();
                actor.setFirstName(rs.getString("first_name"));
                actor.setLastName(rs.getString("last_name"));
                return actor;
            }
        });
```

如果上面的两段代码实际存在于相同的应用中，建议把`RowMapper`匿名类中重复的代码抽取到单独的类中（通常是一个静态类），方便被DAO方法引用。例如，上面的代码例子更好的写法如下：

```java
public List<Actor> findAllActors() {
    return this.jdbcTemplate.query( "select first_name, last_name from t_actor", new ActorMapper());
}

private static final class ActorMapper implements RowMapper<Actor> {

    public Actor mapRow(ResultSet rs, int rowNum) throws SQLException {
        Actor actor = new Actor();
        actor.setFirstName(rs.getString("first_name"));
        actor.setLastName(rs.getString("last_name"));
        return actor;
    }
}
```

**使用jdbcTemplate实现增删改**

你可以使用`update(..)`方法实现插入，更新和删除操作。参数值可以通过可变参数或者封装在对象内传入。

```java
this.jdbcTemplate.update(
        "insert into t_actor (first_name, last_name) values (?, ?)",
        "Leonor", "Watling");
```

```java
this.jdbcTemplate.update(
        "update t_actor set last_name = ? where id = ?",
        "Banjo", 5276L);
```

```java
this.jdbcTemplate.update(
        "delete from actor where id = ?",
        Long.valueOf(actorId));
```

**其他jdbcTemplate操作**

你可以使用`execute(..)`方法执行任何SQL，甚至是DDL语句。这个方法可以传入回调接口、绑定可变参数数组等。

```java
this.jdbcTemplate.execute("create table mytable (id integer, name varchar(100))");
```

下面的例子调用一段简单的存储过程。更复杂的存储过程支持文档[covered later](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/jdbc.html#jdbc-StoredProcedure)。

```java
this.jdbcTemplate.update(
        "call SUPPORT.REFRESH_ACTORS_SUMMARY(?)",
        Long.valueOf(unionId));
```

**JdbcTemplate 最佳实践**

`JdbcTemplate`实例一旦配置之后是线程安全的。这点很重要因为这样你就能够配置`JdbcTemplate`的单例，然后安全的将其注入到多个DAO中（或者repositories）。`JdbcTemplate`是有状态的，内部存在对`DataSource`的引用，但是这种状态不是会话状态。

使用JdbcTemplate类的常用做法是在你的Spring配置文件里配置好一个`DataSource`，然后将其依赖注入进你的DAO类中（[`NamedParameterJdbcTemplate`](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/jdbc.html#jdbc-NamedParameterJdbcTemplate)也是如此）。`JdbcTemplate`在`DataSource`的Setter方法中被创建。就像如下DAO类的写法一样：

```java
public class JdbcCorporateEventDao implements CorporateEventDao {

    private JdbcTemplate jdbcTemplate;

    public void setDataSource(DataSource dataSource) {
        this.jdbcTemplate = new JdbcTemplate(dataSource);
    }

    // JDBC-backed implementations of the methods on the CorporateEventDao follow...
}
```

相关的配置是这样的：

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <bean id="corporateEventDao" class="com.example.JdbcCorporateEventDao">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
        <property name="driverClassName" value="${jdbc.driverClassName}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>

    <context:property-placeholder location="jdbc.properties"/>

</beans>
```

另一种替代显式配置的方式是使用component-scanning和注解注入。在这个场景下需要添加`@Repository`注解（添加这个注解可以被component-scanning扫描到），同时在`DataSource`的Setter方法上添加`@Autowired`注解：

```java
@Repository
public class JdbcCorporateEventDao implements CorporateEventDao {

    private JdbcTemplate jdbcTemplate;

    @Autowired
    public void setDataSource(DataSource dataSource) {
        this.jdbcTemplate = new JdbcTemplate(dataSource);
    }

    // JDBC-backed implementations of the methods on the CorporateEventDao follow...
}
```

相关的XML配置如下：

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <!-- Scans within the base package of the application for @Component classes to configure as beans -->
    <context:component-scan base-package="org.springframework.docs.test" />

    <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
        <property name="driverClassName" value="${jdbc.driverClassName}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>

    <context:property-placeholder location="jdbc.properties"/>

</beans>
```

如果你使用Spring的`JdbcDaoSupport`类，许多JDBC相关的DAO类都从该类继承过来，这个时候相关子类需要继承JdbcDaoSupport类的`setDataSource(..)`方法。当然你也可以选择不从这个类继承，`JdbcDaoSupport`本身只是提供一些便利性。

无论你选择上面提到的哪种初始方式，当你在执行SQL语句时一般都不需要重新创建`JdbcTemplate`实例。JdbcTemplate一旦被配置后其实例都是线程安全的。当你的应用需要访问多个数据库时你可能也需要多个`JdbcTemplate`实例，相应的也需要多个`DataSources`，同时对应多个`JdbcTemplates`配置。

