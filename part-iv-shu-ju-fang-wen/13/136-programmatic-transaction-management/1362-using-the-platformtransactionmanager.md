### 13.6.2Using the PlatformTransactionManager

You can also use the`org.springframework.transaction.PlatformTransactionManager`directly to manage your transaction. Simply pass the implementation of the`PlatformTransactionManager`you are using to your bean through a bean reference. Then, using the`TransactionDefinition`and`TransactionStatus`objects you can initiate transactions, roll back, and commit.

```java
DefaultTransactionDefinition def = new DefaultTransactionDefinition();
// explicitly setting the transaction name is something that can only be done programmatically
def.setName("SomeTxName");
def.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRED);

TransactionStatus status = txManager.getTransaction(def);
try {
	// execute your business logic here
}
catch (MyException ex) {
	txManager.rollback(status);
	throw ex;
}
txManager.commit(status);
```



