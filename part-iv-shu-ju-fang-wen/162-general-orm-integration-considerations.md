## 16.2General ORM integration considerations

This section highlights considerations that apply to all ORM technologies. The[Section16.3, “Hibernate”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/orm.html#orm-hibernate)section provides more details and also show these features and configurations in a concrete context.

The major goal of Spring’s ORM integration is clear application layering, with any data access and transaction technology, and for loose coupling of application objects. No more business service dependencies on the data access or transaction strategy, no more hard-coded resource lookups, no more hard-to-replace singletons, no more custom service registries. One simple and consistent approach to wiring up application objects, keeping them as reusable and free from container dependencies as possible. All the individual data access features are usable on their own but integrate nicely with Spring’s application context concept, providing XML-based configuration and cross-referencing of plain JavaBean instances that need not be Spring-aware. In a typical Spring application, many important objects are JavaBeans: data access templates, data access objects, transaction managers, business services that use the data access objects and transaction managers, web view resolvers, web controllers that use the business services,and so on.

  


