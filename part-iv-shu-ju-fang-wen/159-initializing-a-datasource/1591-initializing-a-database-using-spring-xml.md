### 15.9.1**使用Spring XML来初始化数据库**

如果你想要初始化一个数据库你可以设置`DataSource` Bean的引用，使用`spring-jdbc`命名空间下的`initialize-database`标记

```java
<jdbc:initialize-database data-source="dataSource">
    <jdbc:script location="classpath:com/foo/sql/db-schema.sql"/>
    <jdbc:script location="classpath:com/foo/sql/db-test-data.sql"/>
</jdbc:initialize-database>
```

上面的例子执行了数据库的两个脚本：第一个脚本创建了一个Schema，第二个往表里插入了一个测试数据集。脚本路径可以使用Spring中ant类型的查找资源的模式（例如`classpath*:/com/foo/**/sql/*-data.sql`）。如果使用了正则表达式，脚本会按照URL或者文件名的词法顺序执行。

默认数据库初始器会无条件执行该脚本。有时你并不想要这么做，例如。你正在执行一个已存在测试数据集的数据库脚本。下面的通用方法会避免不小心删除数据，比如像上面的例子先创建表然后再插入-第一步在表已经存在时会失败掉。

为了能够在创建和删除已有数据方面提供更多的控制，XML命名空间提供了更多的方式。第一个是将初始化设置开启还是关闭。这个可以根据当前环境情况设置（例如用系统变量或者环境属性Bean中获取一个布尔值）例如：

```java
<jdbc:initialize-database data-source="dataSource"
    enabled="#{systemProperties.INITIALIZE_DATABASE}">
    <jdbc:script location="..."/>
</jdbc:initialize-database>
```

第二个选项是控制当前数据的行为，这是为了提高容错性。你能够控制初始器来忽略SQL里面执行脚本的特定错误、例如：

```java
<jdbc:initialize-database data-source="dataSource" ignore-failures="DROPS">
    <jdbc:script location="..."/>
</jdbc:initialize-database>
```

在这个例子中我们声明我们预期有时脚本会执行到一个空的数据库，那么脚本中的`DROP`语句会失败掉。这个失败的SQL `DROP`语句会被忽略，但是其他错误会抛出一个异常。这在你的SQL语句不支持`DROP …​ IF EXISTS`（或类似语法）时比较有用，但是你想要在重新创建时无条件的移除所有的测试数据。在这个案例中第一个脚本通常是一系列`DROP`语句的集合，然后是一系列`Create`语句。

`ignore-failures`选项可以被设置成`NONE `\(默认值\), `DROPS `\(忽略失败的删除\), 或 `ALL `\(忽略所有失败\).

每个语句都应该以`;`隔开，或者`;`字符不存在于所用的脚本的话使用新的一行。你可以全局控制或者单针对一个脚本控制，例如：

```java
<jdbc:initialize-database data-source="dataSource" separator="@@">
    <jdbc:script location="classpath:com/foo/sql/db-schema.sql" separator=";"/>
    <jdbc:script location="classpath:com/foo/sql/db-test-data-1.sql"/>
    <jdbc:script location="classpath:com/foo/sql/db-test-data-2.sql"/>
</jdbc:initialize-database>
```

在这个例子中，两个`test-data`脚本使用`@@`作为语句分隔符，并且只有`db-schema.sql`使用`;`.配置指定默认分隔符是`@@`，并且将`db-schema`脚本内容覆盖默认值。

如果你需要从XML命名空间获取更多控制，你可以直接简单的使用`DataSourceInitializer`，并且在你的应用中将其定义为一个模块

**初始化依赖数据库的其他模块**

大多数应用只需要使用数据库初始器基本不会遇到其他问题了：这些基本在Spring上下文初始化之前不会使用到数据库。如果你的应用不属于这种情况则需要使用阅读余下的内容。

数据库初始器依赖于`Datasource`实例，并且在初始化callback方法中执行脚本（类似于XML bean定义中的`init-method`,组件中的`@Postconstruct`方法，或是在实现了`InitializingBean`类的`afterPropertiesSet()`方法）。如果有其他bean也依赖了同样的数据源同时也在初始化回调模块中使用数据源，那么可能会存在问题因为数据还没有初始化。

一个例子是在应用启动时缓存需要从数据库Load并且初始化数据。

为了解决这个问题你有两个选择：改变你的缓存初始化策略将其移到更后面的阶段，或者确保数据库会首先初始化。

如果你可以控制应用的初始化行为那么第一个选项明显更容易。下面是使用这个方式的一些建议，包括：

* 缓存做懒加载，并且只在第一次访问的时候初始化，这样会提高你应用启动时间。

* 让你的缓存或者其他独立模块通过实现`Lifecycle`或者`SmartLifecycle`接口来初始化缓存。当应用上下文启动时如果`autoStartup`有设置，`SmartLifecycle`会被自动启动，而`Lifecycle `可以在调用`ConfigurableApplicationContext.start()`时手动启动。

* 使用Spring的`ApplicationEvent`或者其他类似的自定义监听事件来触发缓存初始化。`ContextRefreshedEvent`总是在Spring容器加载完毕的时候使用（在所有Bean被初始化之后）。这类事件钩子\(hook\)机制在很多时候非常有用（这也是`SmartLifecycle`的默认工作机制）

第二个选项也可以很容易实现。建议如下：

* 依赖Spring `Beanfatory`的默认行为，Bean会按照注册顺序被初始化。你可以通过在XML配置中设置顺序来指定你应用模块的初始化顺序，确保数据库和数据库初始化会首先被执行。

* 将`Datasource`和业务模块隔离，通过将它们放在各自独立的`ApplicationContext`上下文实例来控制其启动顺序。（例如：父上下文包含`DataSource`，子上下文包含业务模块）。这种结构在Spring Web应用中很常用，同时也是一种通用做法



