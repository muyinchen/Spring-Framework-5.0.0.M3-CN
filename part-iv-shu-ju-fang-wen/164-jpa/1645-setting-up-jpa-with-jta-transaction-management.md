### 16.4.5为JPA配置JTA事务管理

作为`JpaTransactionManager`的替代方案，Spring还允许通过JTA在J2EE环境中或与独立的事务协调器（如Atomikos）进行多资源事务协调。除了用Spring的`JtaTransactionManager`替换`JpaTransactionManager`，还有需要以下一些操作：

* 底层JDBC连接池是需要具备XA功能，并与开发者的事务协调器集成的。这在J2EE环境中很简单，只需通过JNDI导出不同类型的`DataSource`即可。有关导出`DataSource`等详细信息，可以参考应用服务器文档。类似地，独立的事务协调器通常带有特殊的XA集成的`DataSource`实现。

* 需要为JTA配置JPA `EntityManagerFactory`。这是特定于提供程序的，通常通过在`LocalContainerEntityManagerFactoryBean`的特殊属性指定为”jpaProperties”。在使用Hibernate的情况下，这些属性甚至是需要基于特定的版本的;请查阅Hibernate文档以获取详细信息。

* Spring的`HibernateJpaVendorAdapter`会强制执行某些面向Spring的默认设置，例如在Hibernate 5.0中匹配Hibernate自己的默认值的连接释放模式“on-close”，但在5.1 / 5.2中不再存在。对于JTA设置，不要声明`HibernateJpaVendorAdapter`开始，或关闭其`prepareConnection`标志。或者，将Hibernate 5.2的`hibernate.connection.handling_mode`属性设置为`DELAYED_ACQUISITION_AND_RELEASE_AFTER_STATEMENT`以恢复Hibernate自己的默认值。有关WebLogic的相关说明，请参考[Section16.3.7, “Hibernate的虚假应用服务器警告”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/orm.html#orm-hibernate-invalid-jdbc-access-error)一节。

* 或者，可以考虑从应用程序服务器本身获取`EntityManagerFactory`，即通过JNDI查找而不是本地声明的`LocalContainerEntityManagerFactoryBean`。服务器提供的`EntityManagerFactory`可能需要在服务器配置中进行特殊定义，减少了部署的移植性，但是`EntityManagerFactory`将为开箱即用的服务器JTA环境设置。



