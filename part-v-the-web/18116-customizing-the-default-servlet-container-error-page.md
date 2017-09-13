### 18.11.6Customizing the Default Servlet Container Error Page

When the status of the response is set to an error status code and the body of the response is empty, Servlet containers commonly render an HTML formatted error page. To customize the default error page of the container, you can declare an```element in``web.xml\`. Up until Servlet 3, that element had to be mapped to a specific status code or exception type. Starting with Servlet 3 an error page does not need to be mapped, which effectively means the specified location customizes the default Servlet container error page.

```java
<error-page>
	<location>/error</location>
</error-page>
```

Note that the actual location for the error page can be a JSP page or some other URL within the container including one handled through an`@Controller`method:

When writing error information, the status code and the error message set on the`HttpServletResponse`can be accessed through request attributes in a controller:

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

or in a JSP:

```java
<%@ page contentType="application/json" pageEncoding="UTF-8"%>
{
	status:<%=request.getAttribute("javax.servlet.error.status_code") %>,
	reason:<%=request.getAttribute("javax.servlet.error.message") %>
}
```



