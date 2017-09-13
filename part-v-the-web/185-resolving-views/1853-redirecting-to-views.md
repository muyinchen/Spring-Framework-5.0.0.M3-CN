### 18.5.3 Redirecting to Views

As mentioned previously, a controller typically returns a logical view name, which a view resolver resolves to a particular view technology. For view technologies such as JSPs that are processed through the Servlet or JSP engine, this resolution is usually handled through the combination of`InternalResourceViewResolver`and`InternalResourceView`, which issues an internal forward or include via the Servlet API’s`RequestDispatcher.forward(..)`method or`RequestDispatcher.include()`method. For other view technologies, such as FreeMarker, XSLT, and so on, the view itself writes the content directly to the response stream.

It is sometimes desirable to issue an HTTP redirect back to the client, before the view is rendered. This is desirable, for example, when one controller has been called with`POST`data, and the response is actually a delegation to another controller \(for example on a successful form submission\). In this case, a normal internal forward will mean that the other controller will also see the same`POST`data, which is potentially problematic if it can confuse it with other expected data. Another reason to perform a redirect before displaying the result is to eliminate the possibility of the user submitting the form data multiple times. In this scenario, the browser will first send an initial`POST`; it will then receive a response to redirect to a different URL; and finally the browser will perform a subsequent`GET`for the URL named in the redirect response. Thus, from the perspective of the browser, the current page does not reflect the result of a`POST`but rather of a`GET`. The end effect is that there is no way the user can accidentally re-`POST`the same data by performing a refresh. The refresh forces a`GET`of the result page, not a resend of the initial`POST`data.

#### RedirectView

One way to force a redirect as the result of a controller response is for the controller to create and return an instance of Spring’s`RedirectView`. In this case,`DispatcherServlet`does not use the normal view resolution mechanism. Rather because it has been given the \(redirect\) view already, the`DispatcherServlet`simply instructs the view to do its work. The`RedirectView`in turn calls`HttpServletResponse.sendRedirect()`to send an HTTP redirect to the client browser.

If you use`RedirectView`and the view is created by the controller itself, it is recommended that you configure the redirect URL to be injected into the controller so that it is not baked into the controller but configured in the context along with the view names. The[the section called “The redirect: prefix”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-redirecting-redirect-prefix)facilitates this decoupling.

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

