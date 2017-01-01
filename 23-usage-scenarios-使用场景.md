## 2.3 Usage scenarios 使用场景

The building blocks described previously make Spring a logical choice in many scenarios, from embedded applications that run on resource-constrained devices to full-fledged enterprise applications that use Spring’s transaction management functionality and web framework integration.

之前描述的构建块使`Spring`成为许多场景中的逻辑(也就是下意识合理的)选择，从资源受限的嵌入式程序到成熟的企业应用程序都可以使用 `Spring `事务管理功能和 `web `框架集成。

**图2.2。 典型的完整Spring Web应用程序**

![overview full](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/images/overview-full.png)

Spring’s [declarative transaction management features](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/#transaction-declarative) make the web application fully transactional, just as it would be if you used EJB container-managed transactions. All your custom business logic can be implemented with simple POJOs and managed by Spring’s IoC container. Additional services include support for sending email and validation that is independent of the web layer, which lets you choose where to execute validation rules. Spring’s ORM support is integrated with JPA and Hibernate; for example, when using Hibernate, you can continue to use your existing mapping files and standard Hibernate `SessionFactory` configuration. Form controllers seamlessly integrate the web-layer with the domain model, removing the need for `ActionForms` or other classes that transform HTTP parameters to values for your domain model.

`Spring`的声明式事务管理特性使`Web`应用程序具有完全事务性，就像使用EJB容器管理的事务一样。 所有的定制业务逻辑都可以通过简单的`POJO`实现，并由`Spring`的`IoC容器`管理。 其他服务包括独立于`Web`层的发送电子邮件和验证的支持，允许你选择执行验证规则的位置。 `Spring`的`ORM`支持集成了`JPA`和`Hibernate`;(`注：和4的文档比少了JDO`) 例如，当使用`Hibernate`时，可以继续使用现有的映射文件和标准的`Hibernate SessionFactory`配置。 表单控制器将`Web`层与域模型无缝集成，从而无需使用`ActionForms`或其他将`HTTP`参数转换为域模型值的类。(`注：也就是这些参数无须单独创建POJO来表达`)

**图2.3。 Spring中间层使用第三方Web框架**

![overview thirdparty web](http://docs.spring.io/spring/docs/5.0.0.M3/spring-framework-reference/htmlsingle/images/overview-thirdparty-web.png)

Sometimes circumstances do not allow you to completely switch to a different framework. The Spring Framework does *not* force you to use everything within it; it is not an*all-or-nothing* solution. Existing front-ends built with Struts, Tapestry, JSF or other UI frameworks can be integrated with a Spring-based middle-tier, which allows you to use Spring transaction features. You simply need to wire up your business logic using an `ApplicationContext` and use a `WebApplicationContext` to integrate your web layer.

有时情况不允许你完全切换到一个不同的框架。 `Spring`框架不强迫你使用它里面的一切; 它不是一个全有或全无的解决方案。 使用`Struts`，`Tapestry`，`JSF`或其他`UI`框架构建的现有前端可以与基于`Spring`的中间层集成，它允许你使用`Spring`事务功能。 你只需要使用`ApplicationContext`连接你的业务逻辑，并使用`WebApplicationContext`来集成你的`web`层。

**图2.4。 远程使用场景**

![overview remoting](http://docs.spring.io/spring/docs/5.0.0.M3/spring-framework-reference/htmlsingle/images/overview-remoting.png)

When you need to access existing code through web services, you can use Spring’s `Hessian-`, `Rmi-` or `HttpInvokerProxyFactoryBean` classes. Enabling remote access to existing applications is not difficult.

当您需要通过`Web`服务访问现有代码时，可以使用`Spring`的`Hessian-，Rmi-`或`HttpInvokerProxyFactoryBean`类(`注：此处和4的文档对比，4使用的是JaxRpcProxyFactory 类`

`)`。 启用对现有应用程序的远程访问并不困难。

**图2.5。 EJB - 包装现有POJO**

![overview ejb](http://docs.spring.io/spring/docs/5.0.0.M3/spring-framework-reference/htmlsingle/images/overview-ejb.png)

The Spring Framework also provides an [access and abstraction layer](http://docs.spring.io/spring/docs/5.0.0.M3/spring-framework-reference/htmlsingle/#ejb) for Enterprise JavaBeans, enabling you to reuse your existing POJOs and wrap them in stateless session beans for use in scalable, fail-safe web applications that might need declarative security.

`Spring Framework`还为`Enterprise JavaBeans`提供了一个 [访问和抽象层](http://docs.spring.io/spring/docs/5.0.0.M3/spring-framework-reference/htmlsingle/#ejb)，使您可以重用现有的`POJOs`，并将其封装在无状态会话`bean`中，以用于可能需要声明性安全性的扩展即有安全故障的`Web`应用程序中。