### 15.3.6 DriverManagerDataSource

The`DriverManagerDataSource`class is an implementation of the standard`DataSource`interface that configures a plain JDBC driver through bean properties, and returns a new`Connection`every time.

This implementation is useful for test and stand-alone environments outside of a Java EE container, either as a`DataSource`bean in a Spring IoC container, or in conjunction with a simple JNDI environment. Pool-assuming`Connection.close()`calls will simply close the connection, so any`DataSource`-aware persistence code should work. However, using JavaBean-style connection pools such as`commons-dbcp`is so easy, even in a test environment, that it is almost always preferable to use such a connection pool over`DriverManagerDataSource`.

