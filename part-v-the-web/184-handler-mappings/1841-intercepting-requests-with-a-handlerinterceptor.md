### 18.4.1用HandlerInterceptor拦截请求

Spring的处理程序映射机制包括处理程序拦截器，当您想要将特定的功能应用于某些请求时（例如，检查主体），这是非常有用的。

位于处理程序映射中的拦截器必须从`org.springframework.web.servlet`包中实现`HandlerInterceptor`。 这个接口定义了三个方法：`preHandle(..)`在实际的处理器被执行之前被调用; `postHandle(..)`被调用后，处理程序被执行; 并且在完成请求完成之后调用`afterCompletion(..)`。 这三种方法应该提供足够的灵活性来进行各种预处理和后处理。

`preHandle(..)`方法返回一个布尔值。 您可以使用此方法来中断或继续处理执行链。 当此方法返回true时，处理程序执行链将继续; 当它返回`false`时，`DispatcherServlet`假定拦截器本身已经处理了请求（并且例如呈现了适当的视图）并且不继续执行执行链中的其他拦截器和实际处理器。

拦截器可以使用`interceptors`属性进行配置，拦截器属性存在于从`AbstractHandlerMapping`扩展的所有`HandlerMapping`类中。 这在下面的例子中显示：

```java
<beans>
    <bean id="handlerMapping"
            class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping">
        <property name="interceptors">
            <list>
                <ref bean="officeHoursInterceptor"/>
            </list>
        </property>
    </bean>

    <bean id="officeHoursInterceptor"
            class="samples.TimeBasedAccessInterceptor">
        <property name="openingTime" value="9"/>
        <property name="closingTime" value="18"/>
    </bean>
</beans>
```

```java
package samples;

public class TimeBasedAccessInterceptor extends HandlerInterceptorAdapter {

    private int openingTime;
    private int closingTime;

    public void setOpeningTime(int openingTime) {
        this.openingTime = openingTime;
    }

    public void setClosingTime(int closingTime) {
        this.closingTime = closingTime;
    }

    public boolean preHandle(HttpServletRequest request, HttpServletResponse response,
            Object handler) throws Exception {
        Calendar cal = Calendar.getInstance();
        int hour = cal.get(HOUR_OF_DAY);
        if (openingTime <= hour && hour < closingTime) {
            return true;
        }
        response.sendRedirect("http://host.com/outsideOfficeHours.html");
        return false;
    }
}
```

由此映射处理的任何请求都被`TimeBasedAccessInterceptor`拦截。 如果当前时间在办公时间之外，用户被重定向到一个静态的HTML文件，例如说你只能在办公时间访问网站。

| ![](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png) |
| :--- |
| 当使用`RequestMappingHandlerMapping`时，实际的处理程序是`HandlerMethod`的一个实例，它标识了将被调用的特定控制器方法。 |

正如你所看到的，Spring适配器类`HandlerInterceptorAdapter`使扩展`HandlerInterceptorinter`面更容易。

| ![](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/tip.png) |
| :--- |
| 在上面的示例中，配置的拦截器将应用于使用带注解的控制器方法处理的所有请求。 如果要缩小拦截器所应用的URL路径，可以使用MVC名称空间或MVC Java配置，或者声明`MappedInterceptor`类型的bean实例来执行此操作。 请参见[Section18.16.1, “Enabling the MVC Java Config or the MVC XML Namespace”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-config-enable)。 |

请注意，`HandlerInterceptor`的`postHandle`方法并不总是非常适合与`@ResponseBody`和`ResponseEntity`方法一起使用。 在这种情况下`HttpMessageConverter`会在调用`postHandle`之前写入并提交响应，从而无法更改响应，例如添加标题。 相反，应用程序可以实现`ResponseBodyAdvice`，并将其声明为`@ControllerAdvice` bean，或直接在`RequestMappingHandlerAdapter`上进行配置。

##  



