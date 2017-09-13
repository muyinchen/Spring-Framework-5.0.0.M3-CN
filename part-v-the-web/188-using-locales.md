## 18.8 Using locales

Most parts of Spring’s architecture support internationalization, just as the Spring web MVC framework does.`DispatcherServlet`enables you to automatically resolve messages using the client’s locale. This is done with`LocaleResolver`objects.

When a request comes in, the`DispatcherServlet`looks for a locale resolver, and if it finds one it tries to use it to set the locale. Using the`RequestContext.getLocale()`method, you can always retrieve the locale that was resolved by the locale resolver.

In addition to automatic locale resolution, you can also attach an interceptor to the handler mapping \(see[Section 18.4.1, “Intercepting requests with a HandlerInterceptor”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-handlermapping-interceptor)for more information on handler mapping interceptors\) to change the locale under specific circumstances, for example, based on a parameter in the request.

Locale resolvers and interceptors are defined in the`org.springframework.web.servlet.i18n`package and are configured in your application context in the normal way. Here is a selection of the locale resolvers included in Spring.

