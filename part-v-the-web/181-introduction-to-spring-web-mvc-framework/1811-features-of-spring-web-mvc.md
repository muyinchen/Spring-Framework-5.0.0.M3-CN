### 18.1.1Features of Spring Web MVC

**Spring Web Flow**

Spring Web Flow \(SWF\) aims to be the best solution for the management of web application page flow.

SWF integrates with existing frameworks like Spring MVC and JSF, in both Servlet and Portlet environments. If you have a business process \(or processes\) that would benefit from a conversational model as opposed to a purely request model, then SWF may be the solution.

SWF allows you to capture logical page flows as self-contained modules that are reusable in different situations, and as such is ideal for building web application modules that guide the user through controlled navigations that drive business processes.

For more information about SWF, consult the[Spring Web Flow website](http://projects.spring.io/spring-webflow/).

Spring’s web module includes many unique web support features:

* _Clear separation of roles_. Each role — controller, validator, command object, form object, model object,`DispatcherServlet`, handler mapping, view resolver, and so on — can be fulfilled by a specialized object.

* _Powerful and straightforward configuration of both framework and application classes as JavaBeans_. This configuration capability includes easy referencing across contexts, such as from web controllers to business objects and validators.

* _Adaptability, non-intrusiveness, and flexibility._Define any controller method signature you need, possibly using one of the parameter annotations \(such as @RequestParam, @RequestHeader, @PathVariable, and more\) for a given scenario.

* _Reusable business code, no need for duplication_. Use existing business objects as command or form objects instead of mirroring them to extend a particular framework base class.

* _Customizable binding and validation_. Type mismatches as application-level validation errors that keep the offending value, localized date and number binding, and so on instead of String-only form objects with manual parsing and conversion to business objects.

* _Customizable handler mapping and view resolution_. Handler mapping and view resolution strategies range from simple URL-based configuration, to sophisticated, purpose-built resolution strategies. Spring is more flexible than web MVC frameworks that mandate a particular technique.

* _Flexible model transfer_. Model transfer with a name/value`Map`supports easy integration with any view technology.

* _Customizable locale, time zone and theme resolution, support for JSPs with or without Spring tag library, support for JSTL, support for FreeMarker without the need for extra bridges, and so on._

* _A simple yet powerful JSP tag library known as the Spring tag library that provides support for features such as data binding and themes_. The custom tags allow for maximum flexibility in terms of markup code. For information on the tag library descriptor, see the appendix entitled[Chapter40,_spring JSP Tag Library_](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/spring-tld.html)

* _A JSP form tag library, introduced in Spring 2.0, that makes writing forms in JSP pages much easier._For information on the tag library descriptor, see the appendix entitled[Chapter41,_spring-form JSP Tag Library_](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/spring-form-tld.html)

* _Beans whose lifecycle is scoped to the current HTTP request or HTTPSession._This is not a specific feature of Spring MVC itself, but rather of the`WebApplicationContext`container\(s\) that Spring MVC uses. These bean scopes are described in[Section3.5.4, “Request, session, application, and WebSocket scopes”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/beans.html#beans-factory-scopes-other)



