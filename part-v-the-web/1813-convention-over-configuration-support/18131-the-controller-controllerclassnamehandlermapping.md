### 18.13.1The Controller ControllerClassNameHandlerMapping

The`ControllerClassNameHandlerMapping`class is a`HandlerMapping`implementation that uses a convention to determine the mapping between request URLs and the`Controller`instances that are to handle those requests.

Consider the following simple`Controller`implementation. Take special notice of the_name_of the class.

```java
public class ViewShoppingCartController implements Controller {

	public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) {
		// the implementation is not hugely important for this example...
	}

}
```

Here is a snippet from the corresponding Spring Web MVC configuration file:

```java
<bean class="org.springframework.web.servlet.mvc.support.ControllerClassNameHandlerMapping"/>

<bean id="viewShoppingCart" class="x.y.z.ViewShoppingCartController">
	<!-- inject dependencies as required... -->
</bean>
```

The`ControllerClassNameHandlerMapping`finds all of the various handler \(or`Controller`\) beans defined in its application context and strips`Controller`off the name to define its handler mappings. Thus,`ViewShoppingCartController`maps to the`/viewshoppingcart*`request URL.

Let’s look at some more examples so that the central idea becomes immediately familiar. \(Notice all lowercase in the URLs, in contrast to camel-cased`Controller`class names.\)

* `WelcomeController`maps to the`/welcome*`request URL

* `HomeController`maps to the`/home*`request URL

* `IndexController`maps to the`/index*`request URL

* `RegisterController`maps to the`/register*`request URL

In the case of`MultiActionController`handler classes, the mappings generated are slightly more complex. The`Controller`names in the following examples are assumed to be`MultiActionController`implementations:

* `AdminController`maps to the`/admin/*`request URL

* `CatalogController`maps to the`/catalog/*`request URL

If you follow the convention of naming your`Controller`implementations as`xxxController`, the`ControllerClassNameHandlerMapping`saves you the tedium of defining and maintaining a potentially_looooong_`SimpleUrlHandlerMapping`\(or suchlike\).

The`ControllerClassNameHandlerMapping`class extends the`AbstractHandlerMapping`base class so you can define`HandlerInterceptor`instances and everything else just as you would with many other`HandlerMapping`implementations.

