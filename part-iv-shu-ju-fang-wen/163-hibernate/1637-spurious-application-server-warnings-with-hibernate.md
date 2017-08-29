### 16.3.7Hibernate的虚假应用服务器警告

在某些具有非常严格的`XADataSource`实现的JTA环境（目前只有一些WebLogic Server和WebSphere版本）中，当配置Hibernate时，没有考虑到JTA的`PlatformTransactionManager`对象，可能会在应用程序服务器日志中显示虚假警告或异常。这些警告或异常经常描述正在访问的连接不再有效，或者JDBC访问不再有效。这通常可能是因为事务不再有效。例如，这是WebLogic的一个实际异常：

```java
java.sql.SQLException: The transaction is no longer active - status: 'Committed'. No
further JDBC access is allowed within this transaction.
```

开发者可以通过配置令Hibernate意识到Spring中同步的JTA`PlatformTransactionManager`实例的存在，这样即可消除掉前面所说的虚假警告信息。开发者有以下两种选择：

* 如果在应用程序上下文中，开发者已经直接获取了JTA`PlatformTransactionManager`对象（可能是从JNDI到`JndiObjectFactoryBean`或者`<jee：jndi-lookup>`标签），并将其提供给Spring的Jta`TransactionManager`（其中最简单的方法就是指定一个引用bean将此JTA `PlatformTransactionManager`实例定义为`LocalSessionFactoryBean`的jta`TransactionManager`属性的值）。 Spring之后会令`PlatformTransactionManager`\`对象对Hibernate可见。

* 更有可能开发者无法获取JTA`PlatformTransactionManager`实例，因为Spring的`JtaTransactionManager`是可以自己找到该实例的。因此，开发者需要配置Hibernate令其直接查找JTA`PlatformTransactionManager`。开发者可以如Hibernate手册中所述那样通过在Hibernate配置中配置应用程序服务器特定的`TransactionManagerLookup`类来执行此操作。

本节的其余部分描述了在`PlatformTransactionManager`对Hibernate可见和`PlatformTransactionManager`对Hibernate不可见的情况下发生的事件序列:

当Hibernate未配置任何对JTA`PlatformTransactionManager`的进行查找时，JTA事务提交时会发生以下事件：

* JTA事务提交

* Spring的`JtaTransactionManager`与JTA事务同步，所以它被JTA事务管理器通过`afterCompletion`回调调用。

* 在其他活动中，此同步令Spring通过Hibernate的`afterTransactionCompletion`触发回调（用于清除Hibernate缓存），然后在Hibernate Session上调用`close()`，从而令Hibernate尝试`close()`JDBC连接。

* 在某些环境中，因为事务已经提交，应用程序服务器会认为`Connection`不可用，导致`Connection.close()`调用会触发警告或错误。

当Hibernate配置了对JTA`PlatformTransactionManager`进行查找时，JTA事务提交时会发生以下事件：

* JTA事务准备提交

* Spring的`JtaTransactionManager`与JTA事务同步，所以JTA事务管理器通过`beforeCompletion`方法来回调事务。

* Spring确定Hibernate与JTA事务同步，并且行为与前一种情况不同。假设Hibernate `Session`需要关闭，Spring将会关闭它。

* JTA事务提交。

* Hibernate与JTA事务同步，所以JTA事务管理器通过`afterCompletion`方法回调事务，可以正确清除其缓存。



