### 18.13.1Controller ControllerClassNameHandlerMapping

`ControllerClassNameHandlerMapping`类是`HandlerMapping`实现，它使用约定来确定请求URL和处理这些请求的`Controller`实例之间的映射。

考虑以下简单的`Controller`实现。 请特别注意class的名称。

```java
public class ViewShoppingCartController implements Controller {

    public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) {
        // the implementation is not hugely important for this example...
    }

}
```

这里是相应的Spring Web MVC配置文件的一个片段：

```java
<bean class="org.springframework.web.servlet.mvc.support.ControllerClassNameHandlerMapping"/>

<bean id="viewShoppingCart" class="x.y.z.ViewShoppingCartController">
    <!-- 根据需要注入依赖关系... -->
</bean>
```

`ControllerClassNameHandlerMapping`查找在其应用程序上下文中定义的所有各种处理程序（或`Controller`）Bean，并从名称中除去`Controller`以定义其处理程序映射。 因此，`ViewShoppingCartController`映射到`/viewshoppingcart *`请求URL。

让我们来看一些更多的例子，让中心思想变得非常熟悉。 （请注意URL中的所有小写字母，而不是骆驼式的`Controller`类名称。）

* `WelcomeController`映射到`/welcome*`请求URL

* `HomeController`映射到`/home*`请求URL

* `IndexController`映射到`/index*`请求URL

* `RegisterController`映射到`/register*`请求URL

在`MultiActionController`处理程序类的情况下，生成的映射稍微复杂一些。 以下示例中的`Controller`名称假定为`MultiActionController`实现：

* `AdminController`映射到`/admin/*`请求URL

* `CatalogController`映射到`/catalog/*`请求URL

如果按照惯例将`Controller`实现命名为`xxxController`，那么`ControllerClassNameHandlerMapping`将为您节省定义和维护`SimpleUrlHandlerMapping`（或类似的）的繁琐工作。

`ControllerClassNameHandlerMapping`类扩展了`AbstractHandlerMapping`基类，因此您可以像处理许多其他`HandlerMapping`实现一样定义`HandlerInterceptor`实例和其他所有内容。

