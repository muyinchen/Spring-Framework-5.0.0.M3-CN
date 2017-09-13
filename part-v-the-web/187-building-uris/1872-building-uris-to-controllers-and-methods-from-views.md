### 18.7.2Building URIs to Controllers and methods from views

You can also build links to annotated controllers from views such as JSP, Thymeleaf, FreeMarker. This can be done using the`fromMappingName`method in`MvcUriComponentsBuilder`which refers to mappings by name.

Every`@RequestMapping`is assigned a default name based on the capital letters of the class and the full method name. For example, the method`getFoo`in class`FooController`is assigned the name "FC\#getFoo". This strategy can be replaced or customized by creating an instance of`HandlerMethodMappingNamingStrategy`and plugging it into your`RequestMappingHandlerMapping`. The default strategy implementation also looks at the name attribute on`@RequestMapping`and uses that if present. That means if the default mapping name assigned conflicts with another \(e.g. overloaded methods\) you can assign a name explicitly on the`@RequestMapping`.

| ![](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png) |
| :--- |
| The assigned request mapping names are logged at TRACE level on startup. |

The Spring JSP tag library provides a function called`mvcUrl`that can be used to prepare links to controller methods based on this mechanism.

For example given:

```java
@RequestMapping("/people/{id}/addresses")
public class PersonAddressController {

    @RequestMapping("/{country}")
    public HttpEntity getAddress(@PathVariable String country) { ... }
}
```

You can prepare a link from a JSP as follows:

```java
<%@ taglib uri="http://www.springframework.org/tags" prefix="s" %>
...
<a href="${s:mvcUrl('PAC#getAddress').arg(0,'US').buildAndExpand('123')}">Get Address</a>
```

The above example relies on the`mvcUrl`JSP function declared in the Spring tag library \(i.e. META-INF/spring.tld\). For more advanced cases \(e.g. a custom base URL as explained in the previous section\), it is easy to define your own function, or use a custom tag file, in order to use a specific instance of`MvcUriComponentsBuilder`with a custom base URL.

