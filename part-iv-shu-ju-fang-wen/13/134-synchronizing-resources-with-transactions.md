It should now be clear how you create different transaction managers, and how they are linked to related resources that need to be synchronized to transactions \(for example `DataSourceTransactionManager `to a JDBC `DataSource`,`HibernateTransactionManager` to a Hibernate `SessionFactory`, and so forth\). This section describes how the application code, directly or indirectly using a persistence API such as JDBC, Hibernate, or JPA, ensures that these resources are created, reused, and cleaned up properly. The section also discusses how transaction synchronization is triggered \(optionally\) through the relevant `PlatformTransactionManager`

.

