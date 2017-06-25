## 13.1 Introduction to Spring Framework transaction management

Comprehensive transaction support is among the most compelling reasons to use the Spring Framework. The Spring Framework provides a consistent abstraction for transaction management that delivers the following benefits:

* Consistent programming model across different transaction APIs such as Java Transaction API \(JTA\), JDBC, Hibernate, and Java Persistence API \(JPA\).

* Support for[declarative transaction management](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/transaction.html#transaction-declarative).

* Simpler API for[programmatic](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/transaction.html#transaction-programmatic)transaction management than complex transaction APIs such as JTA.

* Excellent integration with Spring’s data access abstractions.

The following sections describe the Spring Framework’s transaction value-adds and technologies. \(The chapter also includes discussions of best practices, application server integration, and solutions to common problems.\)

* [Advantages of the Spring Framework’s transaction support model](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/transaction.html#transaction-motivation)describes_why_you would use the Spring Framework’s transaction abstraction instead of EJB Container-Managed Transactions \(CMT\) or choosing to drive local transactions through a proprietary API such as Hibernate.

* [Understanding the Spring Framework transaction abstraction](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/transaction.html#transaction-strategies)outlines the core classes and describes how to configure and obtain`DataSource`instances from a variety of sources.

* [Synchronizing resources with transactions](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/transaction.html#tx-resource-synchronization)describes how the application code ensures that resources are created, reused, and cleaned up properly.

* [Declarative transaction management](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/transaction.html#transaction-declarative)describes support for declarative transaction management.

* [Programmatic transaction management](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/transaction.html#transaction-programmatic)covers support for programmatic \(that is, explicitly coded\) transaction management.

* [Transaction bound event](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/transaction.html#transaction-event)describes how you could use application events within a transaction.

  


