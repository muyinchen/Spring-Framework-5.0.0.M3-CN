### 13.5.7Transaction propagation

This section describes some semantics of transaction propagation in Spring. Please note that this section is not an introduction to transaction propagation proper; rather it details some of the semantics regarding transaction propagation in Spring.

In Spring-managed transactions, be aware of the difference between_physical_and_logical_transactions, and how the propagation setting applies to this difference.

#### Required

![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/tx_prop_required.png)

PROPAGATION\_REQUIRED

When the propagation setting is`PROPAGATION_REQUIRED`, a_logical_transaction scope is created for each method upon which the setting is applied. Each such logical transaction scope can determine rollback-only status individually, with an outer transaction scope being logically independent from the inner transaction scope. Of course, in case of standard`PROPAGATION_REQUIRED`behavior, all these scopes will be mapped to the same physical transaction. So a rollback-only marker set in the inner transaction scope does affect the outer transaction’s chance to actually commit \(as you would expect it to\).

However, in the case where an inner transaction scope sets the rollback-only marker, the outer transaction has not decided on the rollback itself, and so the rollback \(silently triggered by the inner transaction scope\) is unexpected. A corresponding`UnexpectedRollbackException`is thrown at that point. This is_expected behavior_so that the caller of a transaction can never be misled to assume that a commit was performed when it really was not. So if an inner transaction \(of which the outer caller is not aware\) silently marks a transaction as rollback-only, the outer caller still calls commit. The outer caller needs to receive an`UnexpectedRollbackException`to indicate clearly that a rollback was performed instead.

#### RequiresNew

![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/tx_prop_requires_new.png)

PROPAGATION\_REQUIRES\_NEW

`PROPAGATION_REQUIRES_NEW`, in contrast to`PROPAGATION_REQUIRED`, uses a_completely_independent transaction for each affected transaction scope. In that case, the underlying physical transactions are different and hence can commit or roll back independently, with an outer transaction not affected by an inner transaction’s rollback status.

#### Nested

`PROPAGATION_NESTED`uses a_single_physical transaction with multiple savepoints that it can roll back to. Such partial rollbacks allow an inner transaction scope to trigger a rollback_for its scope_, with the outer transaction being able to continue the physical transaction despite some operations having been rolled back. This setting is typically mapped onto JDBC savepoints, so will only work with JDBC resource transactions. See Spring’s`DataSourceTransactionManager`.

