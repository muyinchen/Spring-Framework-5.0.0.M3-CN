### 18.2.1Special Bean Types In the WebApplicationContext

The Spring`DispatcherServlet`uses special beans to process requests and render the appropriate views. These beans are part of Spring MVC. You can choose which special beans to use by simply configuring one or more of them in the`WebApplicationContext`. However, you don’t need to do that initially since Spring MVC maintains a list of default beans to use if you don’t configure any. More on that in the next section. First see the table below listing the special bean types the`DispatcherServlet`relies on.

**Table18.1.Special bean types in the WebApplicationContext**

| Bean type | Explanation |
| :--- | :--- |
| [HandlerMapping](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-handlermapping) | Maps incoming requests to handlers and a list of pre- and post-processors \(handler interceptors\) based on some criteria the details of which vary by`HandlerMapping`implementation. The most popular implementation supports annotated controllers but other implementations exists as well. |
| HandlerAdapter | Helps the`DispatcherServlet`to invoke a handler mapped to a request regardless of the handler is actually invoked. For example, invoking an annotated controller requires resolving various annotations. Thus the main purpose of a`HandlerAdapter`is to shield the`DispatcherServlet`from such details. |
| [HandlerExceptionResolver](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-exceptionhandlers) | Maps exceptions to views also allowing for more complex exception handling code. |
| [ViewResolver](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-viewresolver) | Resolves logical String-based view names to actual`View`types. |
| [LocaleResolver](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-localeresolver)&[LocaleContextResolver](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-timezone) | Resolves the locale a client is using and possibly their time zone, in order to be able to offer internationalized views |
| [ThemeResolver](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-themeresolver) | Resolves themes your web application can use, for example, to offer personalized layouts |
| [MultipartResolver](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-multipart) | Parses multi-part requests for example to support processing file uploads from HTML forms. |
| [FlashMapManager](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-flash-attributes) | Stores and retrieves the "input" and the "output"`FlashMap`that can be used to pass attributes from one request to another, usually across a redirect. |



