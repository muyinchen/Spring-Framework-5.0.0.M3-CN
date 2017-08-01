### 16.3.7Spurious application server warnings with Hibernate

In some JTA environments with very strict`XADataSource`implementations — currently only some WebLogic Server and WebSphere versions — when Hibernate is configured without regard to the JTA`PlatformTransactionManager`object for that environment, it is possible for spurious warning or exceptions to show up in the application server log. These warnings or exceptions indicate that the connection being accessed is no longer valid, or JDBC access is no longer valid, possibly because the transaction is no longer active. As an example, here is an actual exception from WebLogic:

```java
java.sql.SQLException: The transaction is no longer active - status: 'Committed'. No
further JDBC access is allowed within this transaction.
```

You resolve this warning by simply making Hibernate aware of the JTA`PlatformTransactionManager`instance, to which it will synchronize \(along with Spring\). You have two options for doing this:

* If in your application context you are already directly obtaining the JTA`PlatformTransactionManager`object \(presumably from JNDI through`JndiObjectFactoryBean`or```) and feeding it, for example, to Spring’s``JtaTransactionManager`, then the easiest way is to specify a reference to the bean defining this JTA`PlatformTransactionManager`instance as the value of the`jtaTransactionManager`property for`LocalSessionFactoryBean.\`Spring then makes the object available to Hibernate.

* More likely you do not already have the JTA`PlatformTransactionManager`instance, because Spring’s`JtaTransactionManager`can find it itself. Thus you need to configure Hibernate to look up JTA`PlatformTransactionManager`directly. You do this by configuring an application server- specific`TransactionManagerLookup`class in the Hibernate configuration, as described in the Hibernate manual.

The remainder of this section describes the sequence of events that occur with and without Hibernate’s awareness of the JTA`PlatformTransactionManager`.

When Hibernate is not configured with any awareness of the JTA`PlatformTransactionManager`, the following events occur when a JTA transaction commits:

* The JTA transaction commits.

* Spring’s`JtaTransactionManager`is synchronized to the JTA transaction, so it is called back through an_afterCompletion_callback by the JTA transaction manager.

* Among other activities, this synchronization can trigger a callback by Spring to Hibernate, through Hibernate’s`afterTransactionCompletion`callback \(used to clear the Hibernate cache\), followed by an explicit`close()`call on the Hibernate Session, which causes Hibernate to attempt to`close()`the JDBC Connection.

* In some environments, this`Connection.close()`call then triggers the warning or error, as the application server no longer considers the`Connection`usable at all, because the transaction has already been committed.

When Hibernate is configured with awareness of the JTA`PlatformTransactionManager`, the following events occur when a JTA transaction commits:

* the JTA transaction is ready to commit.

* Spring’s`JtaTransactionManager`is synchronized to the JTA transaction, so the transaction is called back through a_beforeCompletion_callback by the JTA transaction manager.

* Spring is aware that Hibernate itself is synchronized to the JTA transaction, and behaves differently than in the previous scenario. Assuming the Hibernate`Session`needs to be closed at all, Spring will close it now.

* The JTA transaction commits.

* Hibernate is synchronized to the JTA transaction, so the transaction is called back through an_afterCompletion_callback by the JTA transaction manager, and can properly clear its cache.



