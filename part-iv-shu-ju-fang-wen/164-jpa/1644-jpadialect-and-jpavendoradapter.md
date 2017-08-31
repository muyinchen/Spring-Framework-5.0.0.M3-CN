### 16.4.4JpaDialect和JpaVendorAdapter

作为高级功能，`JpaTransactionManager`和`AbstractEntityManagerFactoryBean`的子类支持自定义`JpaDialect`，将其作为Bean传递给`jpaDialect`属性。`JpaDialect`实现可以以供应商特定的方式使能Spring支持的一些高级功能：

* 应用特定的事务语义，如自定义隔离级别或事务超时

* 为基于JDBC的DAO导出事务性JDBC连接

* 从`PersistenceExceptions`到Spring `DataAccessExceptions`的异常转义

这对于特殊的事务语义和异常的高级翻译特别有价值。但是Spring使用的默认实现（`DefaultJpaDialect`）是不提供任何特殊功能的。如果需要上述功能，则必须指定适当的方言才可以。

> 作为一个更广泛的供应商适应设施，主要用于Spring的全功能`LocalContainerEntityManagerFactoryBean`设置，`JpaVendorAdapter`将`JpaDialect`的功能与其他提供者特定的默认设置相结合。指定`HibernateJpaVendorAdapter`或`EclipseLinkJpaVendorAdapter`是分别为Hibernate或EclipseLink自动配置`EntityManagerFactory`设置的最简单方便的方法。但是请注意，这些提供程序适配器主要是为了与Spring驱动的事务管理一起使用而设计的，即为了与`JpaTransactionManager`配合使用的。

有关其操作的更多详细信息以及在Spring的JPA支持中如何使用，请参阅`JpaDialect`和`JpaVendorAdapter`的Javadoc。

