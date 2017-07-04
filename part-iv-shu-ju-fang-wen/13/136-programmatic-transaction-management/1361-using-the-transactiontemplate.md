### 13.6.1Using the TransactionTemplate

The`TransactionTemplate`adopts the same approach as other Spring_templates_such as the`JdbcTemplate`. It uses a callback approach, to free application code from having to do the boilerplate acquisition and release of transactional resources, and results in code that is intention driven, in that the code that is written focuses solely on what the developer wants to do.

| ![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| :--- |
| As you will see in the examples that follow, using the`TransactionTemplate`absolutely couples you to Spring’s transaction infrastructure and APIs. Whether or not programmatic transaction management is suitable for your development needs is a decision that you will have to make yourself. |

Application code that must execute in a transactional context, and that will use the`TransactionTemplate`explicitly, looks like the following. You, as an application developer, write a`TransactionCallback`implementation \(typically expressed as an anonymous inner class\) that contains the code that you need to execute in the context of a transaction. You then pass an instance of your custom`TransactionCallback`to the`execute(..)`method exposed on the`TransactionTemplate`.

```java
public class SimpleService implements Service {

	// single TransactionTemplate shared amongst all methods in this instance
	private final TransactionTemplate transactionTemplate;

	// use constructor-injection to supply the PlatformTransactionManager
	public SimpleService(PlatformTransactionManager transactionManager) {
		Assert.notNull(transactionManager, "The 'transactionManager' argument must not be null.");
		this.transactionTemplate = new TransactionTemplate(transactionManager);
	}

	public Object someServiceMethod() {
		return transactionTemplate.execute(new TransactionCallback() {
			// the code in this method executes in a transactional context
			public Object doInTransaction(TransactionStatus status) {
				updateOperation1();
				return resultOfUpdateOperation2();
			}
		});
	}
}
```

If there is no return value, use the convenient`TransactionCallbackWithoutResult`class with an anonymous class as follows:

```java
transactionTemplate.execute(new TransactionCallbackWithoutResult() {
	protected void doInTransactionWithoutResult(TransactionStatus status) {
		updateOperation1();
		updateOperation2();
	}
});
```

Code within the callback can roll the transaction back by calling the`setRollbackOnly()`method on the supplied`TransactionStatus`object:

```java
transactionTemplate.execute(new TransactionCallbackWithoutResult() {

	protected void doInTransactionWithoutResult(TransactionStatus status) {
		try {
			updateOperation1();
			updateOperation2();
		} catch (SomeBusinessExeption ex) {
			status.setRollbackOnly();
		}
	}
});
```

#### Specifying transaction settings

You can specify transaction settings such as the propagation mode, the isolation level, the timeout, and so forth on the`TransactionTemplate`either programmatically or in configuration.`TransactionTemplate`instances by default have the[default transactional settings](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/transaction.html#transaction-declarative-txadvice-settings). The following example shows the programmatic customization of the transactional settings for a specific`TransactionTemplate:`

```java
public class SimpleService implements Service {

	private final TransactionTemplate transactionTemplate;

	public SimpleService(PlatformTransactionManager transactionManager) {
		Assert.notNull(transactionManager, "The 'transactionManager' argument must not be null.");
		this.transactionTemplate = new TransactionTemplate(transactionManager);

		// the transaction settings can be set here explicitly if so desired
		this.transactionTemplate.setIsolationLevel(TransactionDefinition.ISOLATION_READ_UNCOMMITTED);
		this.transactionTemplate.setTimeout(30); // 30 seconds
		// and so forth...
	}
}
```

The following example defines a`TransactionTemplate`with some custom transactional settings, using Spring XML configuration. The`sharedTransactionTemplate`can then be injected into as many services as are required.

```java
<bean id="sharedTransactionTemplate"
		class="org.springframework.transaction.support.TransactionTemplate">
	<property name="isolationLevelName" value="ISOLATION_READ_UNCOMMITTED"/>
	<property name="timeout" value="30"/>
</bean>"
```

Finally, instances of the`TransactionTemplate`class are threadsafe, in that instances do not maintain any conversational state.`TransactionTemplate`instances_do_however maintain configuration state, so while a number of classes may share a single instance of a`TransactionTemplate`, if a class needs to use a`TransactionTemplate`with different settings \(for example, a different isolation level\), then you need to create two distinct`TransactionTemplate`instances.

