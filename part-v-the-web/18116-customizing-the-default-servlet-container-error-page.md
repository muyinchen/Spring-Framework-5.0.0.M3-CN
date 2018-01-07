### 18.11.6自定义默认Servlet容器错误页面

当响应的状态设置为错误状态码并且响应的主体为空时，Servlet容器通常会呈现HTML格式的错误页面。 要定制容器的默认错误页面，可以在`web.xml`中声明一个`<error-page>`。 直到Servlet 3，该元素必须映射到特定的状态码或异常类型。 从Servlet 3开始，不需要映射错误页面，这实际上意味着指定的位置定制了默认的Servlet容器错误页面。

```java
<error-page>
    <location>/error</location>
</error-page>
```

请注意，错误页面的实际位置可以是JSP页面或容器中的其他URL，包括通过`@Controller`方法处理的URL：

在编写错误信息时，可以通过控制器中的请求属性访问`HttpServletResponse`上设置的状态码和错误消息：

```java
@Controller
public class ErrorController {

    @RequestMapping(path = "/error", produces = MediaType.APPLICATION_JSON_UTF8_VALUE)
    @ResponseBody
    public Map<String, Object> handle(HttpServletRequest request) {

        Map<String, Object> map = new HashMap<String, Object>();
        map.put("status", request.getAttribute("javax.servlet.error.status_code"));
        map.put("reason", request.getAttribute("javax.servlet.error.message"));

        return map;
    }

}
```

或在JSP中：

```java
<%@ page contentType="application/json" pageEncoding="UTF-8"%>
{
    status:<%=request.getAttribute("javax.servlet.error.status_code") %>,
    reason:<%=request.getAttribute("javax.servlet.error.message") %>
}
```



