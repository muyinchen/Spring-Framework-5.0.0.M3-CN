### 18.5.3 重定向到视图

如前所述，控制器通常返回一个逻辑视图名称，视图解析器解析为一个特定的视图技术。 对于通过Servlet或JSP引擎处理的JSP等视图技术，此解决方案通常通过`InternalResourceViewResolver`和`InternalResourceView`的组合来处理，`InternalResourceView`和`InternalResourceView`通过Servlet API的`RequestDispatcher.forward(..)`方法或`RequestDispatcher.include()`方法。 对于其他视图技术，如FreeMarker，XSLT等，视图本身直接将内容写入响应流。

在呈现视图之前，有时需要发送HTTP重定向回客户端。例如，当使用`POST`数据调用一个控制器时，这是可取的，而响应实际上是对另一个控制器的委托（例如成功的表单提交）。在这种情况下，正常的内部转发意味着另一个控制器也会看到相同的`POST`数据，如果可能会将其与其他预期数据混淆，则这可能会造成问题。在显示结果之前执行重定向的另一个原因是消除了用户多次提交表单数据的可能性。在这种情况下，浏览器将首先发送一个初始`POST`;它会接收到重定向到不同的URL的响应;最后浏览器会对重定向响应中的URL进行后续的`GET`操作。因此，从浏览器的角度来看，当前页面并不反映`POST`的结果，而是`GET`的结果。最终的效果是用户无法通过`POST`执行刷新而意外地重新发布相同的数据。刷新强制结果页面的`GET`，而不是重新发送初始`POST`数据。

#### RedirectView

作为控制器响应的结果，强制重定向的一种方法是让控制器创建并返回Spring的`RedirectView`实例。 在这种情况下，`DispatcherServlet`不使用普通视图解析机制。 而是因为它已经被赋予了（重定向）视图，`DispatcherServlet`只是指示视图来完成它的工作。 `RedirectView`依次调用`HttpServletResponse.sendRedirect()`向客户端浏览器发送HTTP重定向。

如果使用RedirectView并且视图是由控制器本身创建的，则建议您将重定向URL配置为注入控制器，以使其不会被烘焙到控制器中，而是在上下文中与视图名称一起配置。 在[一节“重定向：前缀”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-redirecting-redirect-prefix)有利于这种脱钩。

##### Passing Data To the Redirect Target

By default all model attributes are considered to be exposed as URI template variables in the redirect URL. Of the remaining attributes those that are primitive types or collections/arrays of primitive types are automatically appended as query parameters.

Appending primitive type attributes as query parameters may be the desired result if a model instance was prepared specifically for the redirect. However, in annotated controllers the model may contain additional attributes added for rendering purposes \(e.g. drop-down field values\). To avoid the possibility of having such attributes appear in the URL, an`@RequestMapping`method can declare an argument of type`RedirectAttributes`and use it to specify the exact attributes to make available to`RedirectView`. If the method does redirect, the content of`RedirectAttributes`is used. Otherwise the content of the model is used.

The`RequestMappingHandlerAdapter`provides a flag called`"ignoreDefaultModelOnRedirect"`that can be used to indicate the content of the default`Model`should never be used if a controller method redirects. Instead the controller method should declare an attribute of type`RedirectAttributes`or if it doesn’t do so no attributes should be passed on to`RedirectView`. Both the MVC namespace and the MVC Java config keep this flag set to`false`in order to maintain backwards compatibility. However, for new applications we recommend setting it to`true`

Note that URI template variables from the present request are automatically made available when expanding a redirect URL and do not need to be added explicitly neither through`Model`nor`RedirectAttributes`. For example:

```java
@PostMapping("/files/{path}")
public String upload(...) {
    // ...
    return "redirect:files/{path}";
}
```

Another way of passing data to the redirect target is via_Flash Attributes_. Unlike other redirect attributes, flash attributes are saved in the HTTP session \(and hence do not appear in the URL\). See[Section18.6, “Using flash attributes”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-flash-attributes)for more information.

#### The redirect: prefix

While the use of`RedirectView`works fine, if the controller itself creates the`RedirectView`, there is no avoiding the fact that the controller is aware that a redirection is happening. This is really suboptimal and couples things too tightly. The controller should not really care about how the response gets handled. In general it should operate only in terms of view names that have been injected into it.

The special`redirect:`prefix allows you to accomplish this. If a view name is returned that has the prefix`redirect:`, the`UrlBasedViewResolver`\(and all subclasses\) will recognize this as a special indication that a redirect is needed. The rest of the view name will be treated as the redirect URL.

The net effect is the same as if the controller had returned a`RedirectView`, but now the controller itself can simply operate in terms of logical view names. A logical view name such as`redirect:/myapp/some/resource`will redirect relative to the current Servlet context, while a name such as`redirect:http://myhost.com/some/arbitrary/path`will redirect to an absolute URL.

Note that the controller handler is annotated with the`@ResponseStatus`, the annotation value takes precedence over the response status set by`RedirectView`.

#### The forward: prefix

It is also possible to use a special`forward:`prefix for view names that are ultimately resolved by`UrlBasedViewResolver`and subclasses. This creates an`InternalResourceView`\(which ultimately does a`RequestDispatcher.forward()`\) around the rest of the view name, which is considered a URL. Therefore, this prefix is not useful with`InternalResourceViewResolver`and`InternalResourceView`\(for JSPs for example\). But the prefix can be helpful when you are primarily using another view technology, but still want to force a forward of a resource to be handled by the Servlet/JSP engine. \(Note that you may also chain multiple view resolvers, instead.\)

As with the`redirect:`prefix, if the view name with the`forward:`prefix is injected into the controller, the controller does not detect that anything special is happening in terms of handling the response.

