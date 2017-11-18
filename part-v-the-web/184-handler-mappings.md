## 18.4处理程序映射

在以前的Spring版本中，用户需要在Web应用程序上下文中定义一个或多个`HandlerMapping` bean，以将传入的Web请求映射到适当的处理程序。 随着注解控制器的引入，您通常不需要这样做，因为`RequestMappingHandlerMapping`自动在所有`@Controller` bean上查找`@RequestMapping`注解。 但请记住，从`AbstractHandlerMapping`扩展的所有`HandlerMapping`类都具有以下可用于自定义其行为的属性：

* `interceptors`使用的拦截器列表。`HandlerInterceptor`在[Section18.4.1, “Intercepting requests with a HandlerInterceptor”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-handlermapping-interceptor)中讨论。

* `defaultHandler`当此处理程序映射不会导致匹配处理程序时使用的缺省处理程序。

* order基于order属性的值（请参阅`org.springframework.core.Ordered`接口），Spring对上下文中可用的所有处理程序映射进行排序，并应用第一个匹配的处理程序。

* `alwaysUseFullPath`如果为`true`，则Spring将使用当前Servlet上下文中的完整路径来查找合适的处理程序。 如果为`false`（默认），则使用当前Servlet映射中的路径。 例如，如果使用`/testing/*`映射Servlet，并且`alwaysUseFullPath`属性设置为`true`，则使用`/testing/viewPage.html`，而如果该属性设置为`false`，则使用`/viewPage.html`。

* 从Spring 2.5开始，`urlDecode`默认为`true`。 如果您想比较编码路径，请将此标志设置为`false`。 但是，`HttpServletRequest`总是以解码形式公开Servlet路径。 请注意，与编码路径相比，Servlet路径不匹配。

以下示例显示如何配置拦截器：

```java
<beans>
    <bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping">
        <property name="interceptors">
            <bean class="example.MyInterceptor"/>
        </property>
    </bean>
</beans>
```



