### 16.4.4JpaDialect and JpaVendorAdapter

As an advanced feature`JpaTransactionManager`and subclasses of`AbstractEntityManagerFactoryBean`support a custom`JpaDialect`, to be passed into the`jpaDialect`bean property. A`JpaDialect`implementation can enable some advanced features supported by Spring, usually in a vendor-specific manner:

* Applying specific transaction semantics such as custom isolation level or transaction timeout\)

* Retrieving the transactional JDBC`Connection`for exposure to JDBC-based DAOs\)

* Advanced translation of`PersistenceExceptions`to Spring`DataAccessExceptions`

This is particularly valuable for special transaction semantics and for advanced translation of exception. The default implementation used \(`DefaultJpaDialect`\) does not provide any special capabilities and if the above features are required, you have to specify the appropriate dialect.

