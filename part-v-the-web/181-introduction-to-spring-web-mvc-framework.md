## 18.1Spring Web MVC框架的介绍

Spring Web模型视图控制器（MVC）框架是围绕一个`DispatcherServlet`设计的，它将请求分派给处理程序，具有可配置的处理程序映射，视图解析，区域设置，本地化和主题解析，并且支持上传文件。默认的处理是基于注解`@Controller`和`@RequestMapping`，提供一系列灵活的处理方法。随着Spring 3.0的推出，通过`@PathVariable`或者其他注解，`@Controller`机制开始允许你去创建 Rest风格的web站点和应用。

在Spring Web MVC 和 Spring中一条关键的准则是“_对扩展开放，对修改关闭_”

在Spring Web MVC中一些核心类的方法被标注为`final`，由于开发者不能用自已的方法去覆盖这些方法，这并不是任意的，而是特别考虑到这个原则。

对于这个准则的解释，请参考Seth Ladd的Expert Spring Web MVC和Web Flow; 具体参见第一版第117页的“A Look At Design”一节。 或者参见

* [Bob Martin, The Open-Closed Principle \(PDF\)](https://www.cs.duke.edu/courses/fall07/cps108/papers/ocp.pdf)

Spring’s view resolution is extremely flexible. A`Controller`is typically responsible for preparing a model`Map`with data and selecting a view name but it can also write directly to the response stream and complete the request. View name resolution is highly configurable through file extension or Accept header content type negotiation, through bean names, a properties file, or even a custom`ViewResolver`implementation. The model \(the M in MVC\) is a`Map`interface, which allows for the complete abstraction of the view technology. You can integrate directly with template based rendering technologies such as JSP and FreeMarker, or directly generate XML, JSON, Atom, and many other types of content. The model`Map`is simply transformed into an appropriate format, such as JSP request attributes or a FreeMarker template model.

当你使用Spring MVC时，你不能在final方法增加切面。例如，你不能在`AbstractController.setSynchronizeOnSession()`增加切面，有关AOP 代理的更多信息以及为什么不能再Final方法增加切面，查看第7.6.1节[“了解AOP代理”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/aop.html#aop-understanding-aop-proxies)。

在Spring Web MVC中，您可以使用任何对象作为命令或表单支持对象;你不需要实现一个特别架构接口或者基类。Spring数据绑定非常灵活：例如，你可以使用程序将类型不匹配当作验证错误而不是系统错误。 因此，您不需要将您的业务对象的属性复制为简单的无格式的字符串，仅用于处理无效提交，或者正确转换字符串。 相反，通常最好直接绑定到您的业务对象。

Spring 的视图处理也是相当灵活，控制器通常负责准备具有数据和选择视图名称的模型映射，但它也可以直接写入响应流并完成请求。视图名称解析可通过文件扩展或Accept标头内容类型协商进行高度配置，通过bean名称，属性文件或甚至自定义的`ViewResolver`实现。模型（MVC中的M）是一个`Map`接口，可以完全提取视图技术，你可以直接与基于模板的渲染技术（如JSP和FreeMarker）集成，或直接生成XML，JSON，Atom和许多其他类型的内容。 模型`Map`可以简单地转换成适当的格式，如JSP请求属性或FreeMarker模板模型。

