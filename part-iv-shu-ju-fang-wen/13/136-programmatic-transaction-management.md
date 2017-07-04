## 13.6Programmatic transaction management

The Spring Framework provides two means of programmatic transaction management:

* Using the`TransactionTemplate`.

* Using a`PlatformTransactionManager`implementation directly.

The Spring team generally recommends the`TransactionTemplate`for programmatic transaction management. The second approach is similar to using the JTA`UserTransaction`API, although exception handling is less cumbersome.

