### 15.3.5 SingleConnectionDataSource

The`SingleConnectionDataSource`class is an implementation of the`SmartDataSource`interface that wraps a_single_`Connection`that is_not_closed after each use. Obviously, this is not multi-threading capable.

If any client code calls`close`in the assumption of a pooled connection, as when using persistence tools, set the`suppressClose`property to`true`. This setting returns a close-suppressing proxy wrapping the physical connection. Be aware that you will not be able to cast this to a native Oracle`Connection`or the like anymore.

This is primarily a test class. For example, it enables easy testing of code outside an application server, in conjunction with a simple JNDI environment. In contrast to`DriverManagerDataSource`, it reuses the same connection all the time, avoiding excessive creation of physical connections.

