### 13.9.2Oracle WebLogic Server

On WebLogic Server 9.0 or above, you typically would use the`WebLogicJtaTransactionManager`instead of the stock`JtaTransactionManager`class. This special WebLogic-specific subclass of the normal`JtaTransactionManager`supports the full power of Spring’s transaction definitions in a WebLogic-managed transaction environment, beyond standard JTA semantics: Features include transaction names, per-transaction isolation levels, and proper resuming of transactions in all cases.

