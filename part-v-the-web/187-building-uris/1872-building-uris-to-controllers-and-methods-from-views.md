### 18.7.2构建来自视图的控制器和方法的URI

您还可以从视图（如JSP，Thymeleaf，FreeMarker）建立到注解控制器的链接。 这可以使用`MvcUriComponentsBuilder`中的`fromMappingName`方法来完成，该方法引用按名称映射。

每个`@RequestMapping`被分配一个基于类的大写字母和完整的方法名称的默认名称。 例如，类`FooController`中的方法`getFoo`被分配名称“FC＃getFoo”。 可以通过创建`HandlerMethodMappingNamingStrategy`实例并将其插入到`RequestMappingHandlerMapping`中来替换或定制此策略。 默认策略实现也查看`@RequestMapping`上的名称属性，并使用它（如果存在）。 这意味着，如果分配的默认映射名称与另一个映射名称（例如重载的方法）冲突，则可以在`@RequestMapping`上明确指定一个名称。

| ![](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png) |
| :--- |
| 分配的请求映射名称在启动时将记录在TRACE级别。 |

Spring JSP标记库提供了一个名为`mvcUrl`的函数，可用于根据此机制来准备控制器方法的链接。

例如给出：

```java
@RequestMapping("/people/{id}/addresses")
public class PersonAddressController {

    @RequestMapping("/{country}")
    public HttpEntity getAddress(@PathVariable String country) { ... }
}
```

您可以从JSP准备一个链接，如下所示：

```java
<%@ taglib uri="http://www.springframework.org/tags" prefix="s" %>
...
<a href="${s:mvcUrl('PAC#getAddress').arg(0,'US').buildAndExpand('123')}">Get Address</a>
```

上面的例子依赖于Spring标记库中声明的`mvcUrl` JSP函数（即META-INF / spring.tld）。 对于更高级的情况（例如上一节中介绍的自定义基本URL），可以很容易地定义自己的函数或使用自定义标记文件，以便使用具有自定义基本URL的特定`MvcUriComponentsBuilder`实例。

