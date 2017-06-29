### 13.5.3Rolling back a declarative transaction

The previous section outlined the basics of how to specify transactional settings for classes, typically service layer classes, declaratively in your application. This section describes how you can control the rollback of transactions in a simple declarative fashion.

The recommended way to indicate to the Spring Framework’s transaction infrastructure that a transaction’s work is to be rolled back is to throw an`Exception`from code that is currently executing in the context of a transaction. The Spring Framework’s transaction infrastructure code will catch any unhandled`Exception`as it bubbles up the call stack, and make a determination whether to mark the transaction for rollback.

In its default configuration, the Spring Framework’s transaction infrastructure code_only_marks a transaction for rollback in the case of runtime, unchecked exceptions; that is, when the thrown exception is an instance or subclass of`RuntimeException`. \(`Error`s will also - by default - result in a rollback\). Checked exceptions that are thrown from a transactional method do_not_result in rollback in the default configuration.

You can configure exactly which`Exception`types mark a transaction for rollback, including checked exceptions. The following XML snippet demonstrates how you configure rollback for a checked, application-specific`Exception`type.

```java
<tx:advice id="txAdvice" transaction-manager="txManager">
	<tx:attributes>
	<tx:method name="get*" read-only="true" rollback-for="NoProductInStockException"/>
	<tx:method name="*"/>
	</tx:attributes>
</tx:advice>
```

You can also specify 'no rollback rules', if you do_not_want a transaction rolled back when an exception is thrown. The following example tells the Spring Framework’s transaction infrastructure to commit the attendant transaction even in the face of an unhandled`InstrumentNotFoundException`.

```java
<tx:advice id="txAdvice">
	<tx:attributes>
	<tx:method name="updateStock" no-rollback-for="InstrumentNotFoundException"/>
	<tx:method name="*"/>
	</tx:attributes>
</tx:advice>
```

When the Spring Framework’s transaction infrastructure catches an exception and is consults configured rollback rules to determine whether to mark the transaction for rollback, the_strongest_matching rule wins. So in the case of the following configuration, any exception other than an`InstrumentNotFoundException`results in a rollback of the attendant transaction.

```java
<tx:advice id="txAdvice">
	<tx:attributes>
	<tx:method name="*" rollback-for="Throwable" no-rollback-for="InstrumentNotFoundException"/>
	</tx:attributes>
</tx:advice>
```

You can also indicate a required rollback_programmatically_. Although very simple, this process is quite invasive, and tightly couples your code to the Spring Framework’s transaction infrastructure:

```java
public void resolvePosition() {
	try {
		// some business logic...
	} catch (NoProductInStockException ex) {
		// trigger rollback programmatically
		TransactionAspectSupport.currentTransactionStatus().setRollbackOnly();
	}
}
```

You are strongly encouraged to use the declarative approach to rollback if at all possible. Programmatic rollback is available should you absolutely need it, but its usage flies in the face of achieving a clean POJO-based architecture.

