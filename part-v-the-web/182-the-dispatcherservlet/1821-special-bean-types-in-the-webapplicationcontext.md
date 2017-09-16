### 18.2.1**WebApplicationContext中的特殊Bean类型**

Spring `DispatcherServlet`使用特殊的bean来处理请求并呈现适当的视图。 这些bean是Spring MVC的一部分。 您可以通过在`WebApplicationContext`中简单配置一个或多个选择要使用的特殊bean。 但是，您最初不需要这样做，因为Spring MVC维护一个默认bean列表，如果您没有配置任何内容。 更多的在下一节。 首先看下表列出`DispatcherServlet`依赖的特殊bean类型。

**Table18.1.**在 WebApplicationContext中的特殊bean类型

| Bean type | Explanation |
| :--- | :--- |
| [HandlerMapping](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-handlermapping) | 根据一些标准将传入的请求映射到处理程序和前处理程序和后处理程序列表（处理程序拦截器），其细节由`HandlerMapping`实现而异。 最流行的实现支持注释控制器，但其他实现也存在。 |
| HandlerAdapter | 帮助`DispatcherServlet`调用映射到请求的处理程序，而不管实际调用哪个处理程序。 例如，调用带注释的控制器需要解析各种注释。 因此，`HandlerAdapter`的主要目的是屏蔽`DispatcherServlet`和这些细节 |
| [HandlerExceptionResolver](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-exceptionhandlers) | 映射视图的异常，也允许更复杂的异常处理代码。 |
| [ViewResolver](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-viewresolver) | 将基于逻辑字符串的视图名称解析为实际的View类型。 |
| [LocaleResolver](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-localeresolver)&[LocaleContextResolver](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-timezone) | 解决客户端正在使用的区域设置以及可能的时区，以便能够提供国际化的视图 |
| [ThemeResolver](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-themeresolver) | 解决您的Web应用程序可以使用的主题，例如，提供个性化的布局 |
| [MultipartResolver](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-multipart) | 解析multi-part请求，以支持从HTML表单处理文件上传。 |
| [FlashMapManager](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-flash-attributes) | 存储并检索可以用于将属性从一个请求传递到另一个请求的“输入”和“输出”FlashMap，通常是通过重定向。 |



