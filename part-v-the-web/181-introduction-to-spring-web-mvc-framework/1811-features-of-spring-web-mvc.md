### 18.1.1Spring Web MVC的特点

**Spring Web 流程**

Spring Web 流程 \(SWF\)的目的是成为最好的Web页面应用流程管理方案，SWF与Servlet 和Portlet 环境中的Spring MVC和JSF等现有框架集成。如果你有一个这样的业务流程，使用会话模型比纯粹的请求要优，那么SWF可能是一个选择。

SWF允许您将逻辑页面流作为在不同情况下可重用的自包含模块捕获，因此非常适合构建引导用户通过驱动业务流程的受控导航的Web应用程序模块。

更多关于SWF的信息，请点击[Spring Web Flow website](http://projects.spring.io/spring-webflow/).

Spring 的Web模块包含许多独特的web支持特性：

* _明确并分离的角色.每个角色_-控制器，验证器，命令对象，构建对象，模型对象，分发器，映射处理器，视图解析等等都是完全的一个特定对象

* _框架和应用程序类作为JavaBeans的强大而直接的配置。_ 此配置功能包括跨上下文的简单引用，例如从Web控制器到业务对象和验证器。

* 可适配，无入侵，灵活，定义您需要的任何控制器方法签名，可能使用给定方案的参数注释之一（例如`@RequestParam`，`@RequestHeader`，`@PathVariable`等）。

* _可重用的业务代码_，不需要重复，使用现有的业务对象作为命令或表单对象，而不是仿照它们来扩展特定的框架基类。

* _自定义绑定和验证_，类型不匹配作为应用程序级验证错误，保持违规值，本地化日期和数字绑定等，而不是只使用仅包含字符串的表单对象进行手动解析和转换为业务对象。

* _自定义的处理程序映射和视图解析_，从简单的URL配置策略到复杂的，特制的策略，Spring比Web MVC框架更灵活，这些框架需要特定的技术。

* 灵活的模型转换，具有名称/值的模型传输Map支持与任何视图技术的轻松集成。

* 本地，时区，主题自定义，支持具有或不具有Spring标签库的JSP，支持JSTL，支持FreeMarker而不需要额外的网桥等等。

* _一个简单而强大的JSP标签库，被称为Spring标签库，为数据绑定和主题等功能提供支持。_ 自定义标签允许在标记代码方面具有最大的灵活性。 有关标签库描述符的信息，请参见附录[Chapter 40,spring JSP Tag Library](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/spring-tld.html)

* 在Spring 2.0中引入的JSP表单标签库，使得在JSP页面中的写入表单更容易。 有关标签库描述符的信息，请参见附录[Chapter 41,spring-form JSP Tag Library](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/spring-form-tld.html)

* Bean的生命周期范围限定在当前的HTTP请求或HTTP Session中。 这不是Spring MVC本身的一个特定功能，而是Spring MVC使用的WebApplicationContext容器。 这些bean范围在[Section 3.5.4, “Request, session, application, and WebSocket scopes”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/beans.html#beans-factory-scopes-other)



