### 18.2.2 默认DispatcherServlet 配置



In[Section 18.16, “Configuring Spring MVC”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-config)you’ll learn about other options for configuring Spring MVC including MVC Java config and the MVC XML namespace both of which provide a simple starting point and assume little knowledge of how Spring MVC works. Regardless of how you choose to configure your application, the concepts explained in this section are fundamental and should be of help to you.

如上一节中针对每个特殊bean所述，`DispatcherServlet`会维护默认使用的实现列表。此信息保存在包`org.springframework.web.servlet`中的文件`DispatcherServlet.properties`中。

所有特殊bean都有一些合理的默认值。不久之后，您将需要自定义这些bean提供的一个或多个属性。例如，将`InternalResourceViewResolver`设置的`prefix`属性配置为视图文件的父位置是很常见的。

无论细节如何，在这里了解的重要概念是，一旦您在`WebApplicationContext`中配置了一个特殊的bean（如`InternalResourceViewResolver`），您可以有效地覆盖该特殊bean类型所使用的默认实现列表。例如，如果配置了`InternalResourceViewResolver`，则会忽略`ViewResolver`实现的默认列表。

在[第18.16节“配置Spring MVC”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-config)中，您将了解配置Spring MVC的其他选项，包括MVC Java配置和MVC XML命名空间，这两者都提供了一个简单的起点，并且对Spring MVC的工作原理几乎不了解。无论您如何选择配置应用程序，本节中介绍的概念都是基础的，应该对您有所帮助。

