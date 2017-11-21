### 18.5.3 重定向到视图

如前所述，控制器通常返回一个逻辑视图名称，视图解析器解析为一个特定的视图技术。 对于通过Servlet或JSP引擎处理的JSP等视图技术，此解决方案通常通过`InternalResourceViewResolver`和`InternalResourceView`的组合来处理，`InternalResourceView`和`InternalResourceView`通过Servlet API的`RequestDispatcher.forward(..)`方法或`RequestDispatcher.include()`方法。 对于其他视图技术，如FreeMarker，XSLT等，视图本身直接将内容写入响应流。

在呈现视图之前，有时需要发送HTTP重定向回客户端。例如，当使用`POST`数据调用一个控制器时，这是可取的，而响应实际上是对另一个控制器的委托（例如成功的表单提交）。在这种情况下，正常的内部转发意味着另一个控制器也会看到相同的`POST`数据，如果可能会将其与其他预期数据混淆，则这可能会造成问题。在显示结果之前执行重定向的另一个原因是消除了用户多次提交表单数据的可能性。在这种情况下，浏览器将首先发送一个初始`POST`;它会接收到重定向到不同的URL的响应;最后浏览器会对重定向响应中的URL进行后续的`GET`操作。因此，从浏览器的角度来看，当前页面并不反映`POST`的结果，而是`GET`的结果。最终的效果是用户无法通过`POST`执行刷新而意外地重新发布相同的数据。刷新强制结果页面的`GET`，而不是重新发送初始`POST`数据。

#### RedirectView

作为控制器响应的结果，强制重定向的一种方法是让控制器创建并返回Spring的`RedirectView`实例。 在这种情况下，`DispatcherServlet`不使用普通视图解析机制。 而是因为它已经被赋予了（重定向）视图，`DispatcherServlet`只是指示视图来完成它的工作。 `RedirectView`依次调用`HttpServletResponse.sendRedirect()`向客户端浏览器发送HTTP重定向。

如果使用RedirectView并且视图是由控制器本身创建的，则建议您将重定向URL配置为注入控制器，以使其不会被烘焙到控制器中，而是在上下文中与视图名称一起配置。 在[一节“重定向：前缀”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-redirecting-redirect-prefix)有利于这种脱钩。

##### 将数据传递到重定向目标

默认情况下，所有的模型属性都被认为是作为重定向URL中的URI模板变量公开的。 其余属性是原始类型或集合/基本类型数组，都自动附加为查询参数。

如果为重定向专门准备了模型实例，那么将基元类型属性附加为查询参数可能是期望的结果。 但是，在带注解的控制器中，模型可能包含为渲染目的而添加的附加属性（例如，下拉字段值）。 为了避免在URL中出现这样的属性，`@RequestMapping`方法可以声明`RedirectAttributes`类型的参数，并使用它来指定可用于`RedirectView`的确切属性。 如果方法确实重定向，则使用`RedirectAttributes`的内容。 否则使用模型的内容。

如果为重定向准备了模型实例，则将原始类型属性作为查询参数附加可能是所需的结果。然而，在注解控制器中，模型可能包含为渲染目的添加的附加属性（例如下拉字段值）。为了避免这种属性出现在URL中的可能性，一种`@RequestMapping`方法可以声明一个类型的参数，`RedirectAttributes`并使用它来指定可供使用的确切属性`RedirectView`。如果方法重定向，则使用内容`RedirectAttributes`。否则使用模型的内容。

`RequestMappingHandlerAdapter`提供了一个名为`“ignoreDefaultModelOnRedirect”`的标志，可用于指示默认`Model`的内容，如果控制器方法重定向，则永远不要使用该模型。 相反，控制器方法应声明`RedirectAttributes`类型的属性，或者如果不这样做，则不应将任何属性传递给`RedirectView`。 MVC命名空间和MVC Java配置都将此标志设置为`false`，以保持向后兼容性。 但是，对于新应用程序，我们建议将其设置为`true`

请注意，当前请求中的URI模板变量在扩展重定向URL时自动可用，不需要通过`Model`或`RedirectAttributes`显式添加。 例如：

```java
@PostMapping("/files/{path}")
public String upload(...) {
    // ...
    return "redirect:files/{path}";
}
```

将数据传递到重定向目标的另一种方法是通过_Flash属性_。与其他重定向属性不同，Flash属性保存在HTTP会话中（因此不会出现在URL中）。有关详细信息，请参见[第18.6节“使用Flash属性”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-flash-attributes)。

#### 重定向：前缀

虽然使用RedirectView可以正常工作，但如果控制器本身创建RedirectView，则不会避免控制器意识到重定向正在发生。 这实在是不理想，让事情太紧密。 控制器不应该在乎如何处理响应。 一般来说，它只能在注入视图名称的情况下运行。

特殊的`redirect:`前缀可以让你做到这一点。 如果返回具有前缀`redirect：`的视图名称，则`UrlBasedViewResolver`（和所有子类）将认识到这是需要重定向的特殊指示。 视图名称的其余部分将被视为重定向URL。

请注意，控制器处理程序使用注释`@ResponseStatus`，注释值优先于设置的响应状态`RedirectView`。

实际效果与控制器返回`RedirectView`的效果相同，但现在控制器本身可以简单地按照逻辑视图名称操作。 逻辑视图名称，例如`redirect:/myapp/some/resource`将相对于当前Servlet上下文重定向，而诸如`redirect:http://myhost.com/some/arbitrary/path`之类的名称将重定向到绝对URL。

请注意，控制器处理程序使用`@ResponseStatus`注解，注解值优先于`RedirectView`设置的响应状态。

#### 转发：前缀

对于最终由`UrlBasedViewResolver`和子类解析的视图名称，也可以使用特殊的`forward：`前缀。 这将创建一个内部`ResourceView`（最终执行一个`RequestDispatcher.forward()`），其余的视图名称被视为一个URL。 因此，这个前缀对于`InternalResourceViewResolver`和`InternalResourceView`（例如JSP）是没有用的。 但是，当您主要使用其他视图技术时，前缀可能会很有帮助，但仍希望强制转发资源，以便由Servlet/JSP引擎处理。 （请注意，您也可以链接多个视图解析器）。

与`redirect:`前缀一样，如果带有`forward：`前缀的视图名称被注入到控制器中，则控制器不会检测到处理响应方面的任何特殊情况。

