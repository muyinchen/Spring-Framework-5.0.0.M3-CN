## 18.8 使用区域设置

Spring体系结构的大部分支持国际化，就像Spring web MVC框架一。`DispatcherServlet`使您能够使用客户端的语言环境自动解析消息。 这是用`LocaleResolver`对象完成的。

当请求进入时，`DispatcherServlet`会查找一个语言环境解析器，如果找到，它会尝试使用它来设置语言环境。 使用`RequestContext.getLocale()`方法，您始终可以检索由区域设置解析程序解析的区域设置。

除了自动区域设置解析之外，您还可以将拦截器附加到处理程序映射（有关处理程序映射拦截器的更多信息，请参见[第18.4.1节“拦截与HandlerInterceptor的请求”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-handlermapping-interceptor)），以在特定情况下更改区域设置，例如，基于 在请求中的一个参数。

区域设置解析器和拦截器在`org.springframework.web.servlet.i18n`包中定义，并以常规方式在应用程序上下文中配置。 这里是Spring中包含的一个语言环境解析器的选择。

