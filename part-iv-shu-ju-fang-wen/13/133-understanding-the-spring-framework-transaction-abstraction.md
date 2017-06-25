## 13.3 Understanding the Spring Framework transaction abstraction

The key to the Spring transaction abstraction is the notion of a_transaction strategy_. A transaction strategy is defined by the`org.springframework.transaction.PlatformTransactionManager`interface:

```java
public interface PlatformTransactionManager {

	TransactionStatus getTransaction(
			TransactionDefinition definition) throws TransactionException;

	void commit(TransactionStatus status) throws TransactionException;

	void rollback(TransactionStatus status) throws TransactionException;
}
```

This is primarily a service provider interface \(SPI\), although it can be used[programmatically](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/transaction.html#transaction-programmatic-ptm)from your application code. Because`PlatformTransactionManager`is an_interface_, it can be easily mocked or stubbed as necessary. It is not tied to a lookup strategy such as JNDI.`PlatformTransactionManager`implementations are defined like any other object \(or bean\) in the Spring Framework IoC container. This benefit alone makes Spring Framework transactions a worthwhile abstraction even when you work with JTA. Transactional code can be tested much more easily than if it used JTA directly.

Again in keeping with Spring’s philosophy, the`TransactionException`that can be thrown by any of the`PlatformTransactionManager`interface’s methods is_unchecked_\(that is, it extends the`java.lang.RuntimeException`class\). Transaction infrastructure failures are almost invariably fatal. In rare cases where application code can actually recover from a transaction failure, the application developer can still choose to catch and handle`TransactionException`. The salient point is that developers are not_forced_to do so.

The`getTransaction(..)`method returns a`TransactionStatus`object, depending on a`TransactionDefinition`parameter. The returned`TransactionStatus`might represent a new transaction, or can represent an existing transaction if a matching transaction exists in the current call stack. The implication in this latter case is that, as with Java EE transaction contexts, a`TransactionStatus`is associated with a_thread_of execution.

The`TransactionDefinition`interface specifies:

* _Isolation_
  : The degree to which this transaction is isolated from the work of other transactions. For example, can this transaction see uncommitted writes from other transactions?
* _Propagation_
  : Typically, all code executed within a transaction scope will run in that transaction. However, you have the option of specifying the behavior in the event that a transactional method is executed when a transaction context already exists. For example, code can continue running in the existing transaction \(the common case\); or the existing transaction can be suspended and a new transaction created.
  _Spring offers all of the transaction propagation options familiar from EJB CMT_
  . To read about the semantics of transaction propagation in Spring, see
  [Section 13.5.7, “Transaction propagation”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/transaction.html#tx-propagation)
  .
* _Timeout_
  : How long this transaction runs before timing out and being rolled back automatically by the underlying transaction infrastructure.
* _Read-only status_
  : A read-only transaction can be used when your code reads but does not modify data. Read-only transactions can be a useful optimization in some cases, such as when you are using Hibernate.

These settings reflect standard transactional concepts. If necessary, refer to resources that discuss transaction isolation levels and other core transaction concepts. Understanding these concepts is essential to using the Spring Framework or any transaction management solution.

The`TransactionStatus`interface provides a simple way for transactional code to control transaction execution and query transaction status. The concepts should be familiar, as they are common to all transaction APIs:

```java
public interface TransactionStatus extends SavepointManager {

	boolean isNewTransaction();

	boolean hasSavepoint();

	void setRollbackOnly();

	boolean isRollbackOnly();

	void flush();

	boolean isCompleted();

}
```

Regardless of whether you opt for declarative or programmatic transaction management in Spring, defining the correct`PlatformTransactionManager`implementation is absolutely essential. You typically define this implementation through dependency injection.

`PlatformTransactionManager`implementations normally require knowledge of the environment in which they work: JDBC, JTA, Hibernate, and so on. The following examples show how you can define a local`PlatformTransactionManager`implementation. \(This example works with plain JDBC.\)

You define a JDBC`DataSource`

```java
<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
	<property name="driverClassName" value="${jdbc.driverClassName}" />
	<property name="url" value="${jdbc.url}" />
	<property name="username" value="${jdbc.username}" />
	<property name="password" value="${jdbc.password}" />
</bean>
```

The related`PlatformTransactionManager`bean definition will then have a reference to the`DataSource`definition. It will look like this:

```java
<bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
	<property name="dataSource" ref="dataSource"/>
</bean>
```

If you use JTA in a Java EE container then you use a container`DataSource`, obtained through JNDI, in conjunction with Spring’s`JtaTransactionManager`. This is what the JTA and JNDI lookup version would look like:

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:jee="http://www.springframework.org/schema/jee"
	xsi:schemaLocation="
		http://www.springframework.org/schema/beans
		http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/jee
		http://www.springframework.org/schema/jee/spring-jee.xsd">

	<jee:jndi-lookup id="dataSource" jndi-name="jdbc/jpetstore"/>

	<bean id="txManager" class="org.springframework.transaction.jta.JtaTransactionManager" />

	<!-- other <bean/> definitions here -->

</beans>
```

The`JtaTransactionManager`does not need to know about the`DataSource`, or any other specific resources, because it uses the container’s global transaction management infrastructure.

| ![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png.pagespeed.ce.9zQ_1wVwzR.png "\[Note\]") |
| :--- |
| The above definition of the`dataSource`bean uses the`<jndi-lookup/>`tag from the`jee`namespace. For more information on schema-based configuration, see[Chapter 38,XML Schema-based configuration](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/xsd-configuration.html), and for more information on the`<jee/>`tags see the section entitled[Section 38.2.3, “the jee schema”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/xsd-configuration.html#xsd-config-body-schemas-jee). |

You can also use Hibernate local transactions easily, as shown in the following examples. In this case, you need to define a Hibernate`LocalSessionFactoryBean`, which your application code will use to obtain Hibernate`Session`instances.

The`DataSource`bean definition will be similar to the local JDBC example shown previously and thus is not shown in the following example.

| ![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png.pagespeed.ce.9zQ_1wVwzR.png "\[Note\]") |
| :--- |


| If the`DataSource`, used by any non-JTA transaction manager, is looked up via JNDI and managed by a Java EE container, then it should be non-transactional because the Spring Framework, rather than the Java EE container, will manage the transactions. |
| :--- |


The`txManager`bean in this case is of the`HibernateTransactionManager`type. In the same way as the`DataSourceTransactionManager`needs a reference to the`DataSource`, the`HibernateTransactionManager`needs a reference to the`SessionFactory`.

```java
<bean id="sessionFactory" class="org.springframework.orm.hibernate5.LocalSessionFactoryBean">
	<property name="dataSource" ref="dataSource"/>
	<property name="mappingResources">
		<list>
			<value>org/springframework/samples/petclinic/hibernate/petclinic.hbm.xml</value>
		</list>
	</property>
	<property name="hibernateProperties">
		<value>
			hibernate.dialect=${hibernate.dialect}
		</value>
	</property>
</bean>

<bean id="txManager" class="org.springframework.orm.hibernate5.HibernateTransactionManager">
	<property name="sessionFactory" ref="sessionFactory"/>
</bean>
```

If you are using Hibernate and Java EE container-managed JTA transactions, then you should simply use the same`JtaTransactionManager`as in the previous JTA example for JDBC.

```java
<bean id="txManager" class="org.springframework.transaction.jta.JtaTransactionManager"/>
```

| ![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png.pagespeed.ce.9zQ_1wVwzR.png "\[Note\]") |
| :--- |
| If you use JTA , then your transaction manager definition will look the same regardless of what data access technology you use, be it JDBC, Hibernate JPA or any other supported technology. This is due to the fact that JTA transactions are global transactions, which can enlist any transactional resource. |

In all these cases, application code does not need to change. You can change how transactions are managed merely by changing configuration, even if that change means moving from local to global transactions or vice versa.

