## 18.4Handler mappings

In previous versions of Spring, users were required to define one or more`HandlerMapping`beans in the web application context to map incoming web requests to appropriate handlers. With the introduction of annotated controllers, you generally don’t need to do that because the`RequestMappingHandlerMapping`automatically looks for`@RequestMapping`annotations on all`@Controller`beans. However, do keep in mind that all`HandlerMapping`classes extending from`AbstractHandlerMapping`have the following properties that you can use to customize their behavior:

* `interceptors`List of interceptors to use.`HandlerInterceptor`s are discussed in[Section18.4.1, “Intercepting requests with a HandlerInterceptor”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-handlermapping-interceptor).

* `defaultHandler`Default handler to use, when this handler mapping does not result in a matching handler.

* `order`Based on the value of the order property \(see the`org.springframework.core.Ordered`interface\), Spring sorts all handler mappings available in the context and applies the first matching handler.

* `alwaysUseFullPath`If`true`, Spring uses the full path within the current Servlet context to find an appropriate handler. If`false`\(the default\), the path within the current Servlet mapping is used. For example, if a Servlet is mapped using`/testing/*`and the`alwaysUseFullPath`property is set to true,`/testing/viewPage.html`is used, whereas if the property is set to false,`/viewPage.html`is used.

* `urlDecode`Defaults to`true`, as of Spring 2.5. If you prefer to compare encoded paths, set this flag to`false`. However, the`HttpServletRequest`always exposes the Servlet path in decoded form. Be aware that the Servlet path will not match when compared with encoded paths.

The following example shows how to configure an interceptor:

```java
<beans>
	<bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping">
		<property name="interceptors">
			<bean class="example.MyInterceptor"/>
		</property>
	</bean>
</beans>
```



