### 13.4.3TransactionAwareDataSourceProxy

At the very lowest level exists the`TransactionAwareDataSourceProxy`class. This is a proxy for a target`DataSource`, which wraps the target`DataSource`to add awareness of Spring-managed transactions. In this respect, it is similar to a transactional JNDI`DataSource`as provided by a Java EE server.

It should almost never be necessary or desirable to use this class, except when existing code must be called and passed a standard JDBC`DataSource`interface implementation. In that case, it is possible that this code is usable, but participating in Spring managed transactions. It is preferable to write your new code by using the higher level abstractions mentioned above.

