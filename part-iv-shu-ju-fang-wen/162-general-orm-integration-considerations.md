## 16.2集成ORM的注意事项

本节重点介绍适用于所有集成ORM技术的注意事项。在[Section16.3, “Hibernate”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/orm.html#orm-hibernate)一节中提供了很多关于如何配置和使用这些特性提的信息。

Spring对ORM集成的主要目的是使应用层次化，可以任意选择数据访问和事务管理技术，并且为应用对象提供松耦合结构。不再将业务逻辑依赖于数据访问或者事务策略上，不再使用基于硬编码的资源查找，不再使用难以替代的单例，不再自定义服务的注册。同时，为应用提供一个简单和一致的方法来装载对象，保证他们的重用并且尽可能不依赖于容器。所有单独的数据访问功能都可以自己使用，也可以很好地与Spring的`ApplicationContext`集成，提供基于XML的配置和不需要Spring感知的普通`JavaBean`实例。在典型的Spring应用程序中，许多重要的对象都是`JavaBean`：数据访问模板，数据访问对象，事务管理器，使用数据访问对象和事务管理器的业务服务，Web视图解析器，使用业务服务的Web控制器等等。

