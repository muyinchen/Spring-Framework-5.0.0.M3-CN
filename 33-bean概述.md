## 3.3 Bean概述

A Spring IoC container manages one or more *beans*. These beans are created with the configuration metadata that you supply to the container, for example, in the form of XML `` definitions.

Within the container itself, these bean definitions are represented as `BeanDefinition` objects, which contain (among other information) the following metadata:

- *A package-qualified class name:* typically the actual implementation class of the bean being defined.
- Bean behavioral configuration elements, which state how the bean should behave in the container (scope, lifecycle callbacks, and so forth).
- References to other beans that are needed for the bean to do its work; these references are also called *collaborators* or *dependencies*.
- Other configuration settings to set in the newly created object, for example, the number of connections to use in a bean that manages a connection pool, or the size limit of the pool.

This metadata translates to a set of properties that make up each bean definition.


Spring IoC容器管理一个或多个* beans *。这些bean是使用您提供给容器的配置元数据创建的，例如，以XML“定义”的形式。

在容器本身内，这些bean定义表示为`BeanDefinition`对象，它包含（除其他信息之外）以下元数据：

- *包限定类名：*通常是定义的bean的实际实现类。
- Bean行为配置元素，它说明了bean在容器中的行为（范围，生命周期回调等等）。
- 引用其他bean的bean需要做的工作;这些引用也称为*协作者*或*依赖性*。
- 在新创建的对象中设置的其他配置设置，例如，在管理连接池的bean中使用的连接数，或者池的大小限制。

此元数据转换为构成每个bean定义的一组属性。

**Table 3.1. The bean definition**
| Property                 | Explained in…​   解释所在地方... ​                           |
| ------------------------ | ---------------------------------------- |
| class                    | [Section 3.3.2, “Instantiating beans”](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/#beans-factory-class) |
| name                     | [Section 3.3.1, “Naming beans”](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/#beans-beanname) |
| scope                    | [Section 3.5, “Bean scopes”](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/#beans-factory-scopes) |
| constructor arguments    | [Section 3.4.1, “Dependency Injection”](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/#beans-factory-collaborators) |
| properties               | [Section 3.4.1, “Dependency Injection”](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/#beans-factory-collaborators) |
| autowiring mode          | [Section 3.4.5, “Autowiring collaborators”](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/#beans-factory-autowire) |
| lazy-initialization mode | [Section 3.4.4, “Lazy-initialized beans”](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/#beans-factory-lazy-init) |
| initialization method    | [the section called “Initialization callbacks”](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/#beans-factory-lifecycle-initializingbean) |
| destruction method       | [the section called “Destruction callbacks”](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/#beans-factory-lifecycle-disposablebean) |



In addition to bean definitions that contain information on how to create a specific bean, the `ApplicationContext` implementations also permit the registration of existing objects that are created outside the container, by users. This is done by accessing the ApplicationContext’s BeanFactory via the method `getBeanFactory()`which returns the BeanFactory implementation `DefaultListableBeanFactory`. `DefaultListableBeanFactory` supports this registration through the methods `registerSingleton(..)` and `registerBeanDefinition(..)`. However, typical applications work solely with beans defined through metadata bean definitions.

除了包含如何创建特定bean的bean定义之外，ApplicationContext实现还允许注册由用户在容器外部创建的现有对象。 这是通过访问ApplicationContext的BeanFactory通过方法`getBeanFactory()`来实现的，它返回BeanFactory实现`DefaultListableBeanFactory`。 `DefaultListableBeanFactory`通过方法`registerSingleton(..)`和`registerBeanDefinition(..)`支持这种注册。 然而,典型的应用程序只能通过元数据定义的 bean 来定义。

| ![[Note]](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/images/note.png) |
| ---------------------------------------- |
| Bean元数据和手动提供的单例实例需要尽早注册，以便容器在自动装配和其他自检步骤期间正确地进行推理。 虽然在某种程度上支持覆盖现有元数据和现有单例实例，但是在运行时（与动态访问工厂同时）对新bean的注册未被官方支持，并且可能导致并发访问异常和/或bean容器中的不一致状态 。 |