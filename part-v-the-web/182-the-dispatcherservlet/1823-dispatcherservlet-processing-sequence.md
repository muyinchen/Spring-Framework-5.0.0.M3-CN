### 18.2.3DispatcherServlet Processing Sequence

After you set up a`DispatcherServlet`, and a request comes in for that specific`DispatcherServlet`, the`DispatcherServlet`starts processing the request as follows:

* The`WebApplicationContext`is searched for and bound in the request as an attribute that the controller and other elements in the process can use. It is bound by default under the key`DispatcherServlet.WEB_APPLICATION_CONTEXT_ATTRIBUTE`.

* The locale resolver is bound to the request to enable elements in the process to resolve the locale to use when processing the request \(rendering the view, preparing data, and so on\). If you do not need locale resolving, you do not need it.

* The theme resolver is bound to the request to let elements such as views determine which theme to use. If you do not use themes, you can ignore it.

* If you specify a multipart file resolver, the request is inspected for multiparts; if multiparts are found, the request is wrapped in a`MultipartHttpServletRequest`for further processing by other elements in the process. See[Section18.10, “Spring’s multipart \(file upload\) support”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-multipart)for further information about multipart handling.

* An appropriate handler is searched for. If a handler is found, the execution chain associated with the handler \(preprocessors, postprocessors, and controllers\) is executed in order to prepare a model or rendering.

* If a model is returned, the view is rendered. If no model is returned, \(may be due to a preprocessor or postprocessor intercepting the request, perhaps for security reasons\), no view is rendered, because the request could already have been fulfilled.

Handler exception resolvers that are declared in the`WebApplicationContext`pick up exceptions that are thrown during processing of the request. Using these exception resolvers allows you to define custom behaviors to address exceptions.

The Spring`DispatcherServlet`also supports the return of the_last-modification-date_, as specified by the Servlet API. The process of determining the last modification date for a specific request is straightforward: the`DispatcherServlet`looks up an appropriate handler mapping and tests whether the handler that is found implements the_LastModified_interface. If so, the value of the`long getLastModified(request)`method of the`LastModified`interface is returned to the client.

You can customize individual`DispatcherServlet`instances by adding Servlet initialization parameters \(`init-param`elements\) to the Servlet declaration in the`web.xml`file. See the following table for the list of supported parameters.

**Table18.2.DispatcherServlet initialization parameters**

| Parameter | Explanation |
| :--- | :--- |
| `contextClass` | Class that implements`WebApplicationContext`, which instantiates the context used by this Servlet. By default, the`XmlWebApplicationContext`is used. |
| `contextConfigLocation` | String that is passed to the context instance \(specified by`contextClass`\) to indicate where context\(s\) can be found. The string consists potentially of multiple strings \(using a comma as a delimiter\) to support multiple contexts. In case of multiple context locations with beans that are defined twice, the latest location takes precedence. |
| `namespace` | Namespace of the`WebApplicationContext`. Defaults to`[servlet-name]-servlet`. |

  


