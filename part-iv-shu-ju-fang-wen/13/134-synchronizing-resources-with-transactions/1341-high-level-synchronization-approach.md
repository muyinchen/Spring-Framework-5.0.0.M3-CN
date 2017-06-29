### 13.4.1High-level synchronization approach

The preferred approach is to use Spring’s highest level template based persistence integration APIs or to use native ORM APIs with transaction- aware factory beans or proxies for managing the native resource factories. These transaction-aware solutions internally handle resource creation and reuse, cleanup, optional transaction synchronization of the resources, and exception mapping. Thus user data access code does not have to address these tasks, but can be focused purely on non-boilerplate persistence logic. Generally, you use the native ORM API or take a_template_approach for JDBC access by using the`JdbcTemplate`. These solutions are detailed in subsequent chapters of this reference documentation.

