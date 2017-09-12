### 18.3.3Defining @RequestMapping handler methods

`@RequestMapping`handler methods can have very flexible signatures. The supported method arguments and return values are described in the following section. Most arguments can be used in arbitrary order with the only exception being`BindingResult`arguments. This is described in the next section.

| ![](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png) |
| :--- |
| Spring 3.1 introduced a new set of support classes for`@RequestMapping`methods called`RequestMappingHandlerMapping`and`RequestMappingHandlerAdapter`respectively. They are recommended for use and even required to take advantage of new features in Spring MVC 3.1 and going forward. The new support classes are enabled by default from the MVC namespace and with use of the MVC Java config but must be configured explicitly if using neither. |

#### Supported method argument types

The following are the supported method arguments:

* `org.springframework.web.context.request.WebRequest`or`org.springframework.web.context.request.NativeWebRequest`. Allows for generic request parameter access as well as request/session attribute access, without ties to the native Servlet API.

* Request or response objects \(Servlet API\). Choose any specific request or response type, for example`ServletRequest`or`HttpServletRequest`or Spring’s`MultipartRequest`/`MultipartHttpServletRequest`.

* Session object \(Servlet API\) of type`HttpSession`. An argument of this type enforces the presence of a corresponding session. As a consequence, such an argument is never`null`.

| ![](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png) |
| :--- |
| Session access may not be thread-safe, in particular in a Servlet environment. Consider setting the`RequestMappingHandlerAdapter`'s "synchronizeOnSession" flag to "true" if multiple requests are allowed to access a session concurrently. |

* `java.servlet.http.PushBuilder`for the associated Servlet 4.0 push builder API, allowing for programmatic HTTP/2 resource pushes.

* `java.security.Principal`\(or a specific`Principal`implementation class if known\), containing the currently authenticated user.

* `org.springframework.http.HttpMethod`for the HTTP request method, represented as Spring’s`HttpMethod`enum.

* `java.util.Locale`for the current request locale, determined by the most specific locale resolver available, in effect, the configured`LocaleResolver`/`LocaleContextResolver`in an MVC environment.

* `java.util.TimeZone`\(Java 6+\) /`java.time.ZoneId`\(Java 8+\) for the time zone associated with the current request, as determined by a`LocaleContextResolver`.

* `java.io.InputStream`/`java.io.Reader`for access to the request’s content. This value is the raw InputStream/Reader as exposed by the Servlet API.

* `java.io.OutputStream`/`java.io.Writer`for generating the response’s content. This value is the raw OutputStream/Writer as exposed by the Servlet API.

* `@PathVariable`annotated parameters for access to URI template variables. See[the section called “URI Template Patterns”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-ann-requestmapping-uri-templates).

* `@MatrixVariable`annotated parameters for access to name-value pairs located in URI path segments. See[the section called “Matrix Variables”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-ann-matrix-variables).

* `@RequestParam`annotated parameters for access to specific Servlet request parameters. Parameter values are converted to the declared method argument type. See[the section called “Binding request parameters to method parameters with @RequestParam”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-ann-requestparam).

* `@RequestHeader`annotated parameters for access to specific Servlet request HTTP headers. Parameter values are converted to the declared method argument type. See[the section called “Mapping request header attributes with the @RequestHeader annotation”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-ann-requestheader).

* `@RequestBody`annotated parameters for access to the HTTP request body. Parameter values are converted to the declared method argument type using`HttpMessageConverter`s. See[the section called “Mapping the request body with the @RequestBody annotation”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-ann-requestbody).

* `@RequestPart`annotated parameters for access to the content of a "multipart/form-data" request part. See[Section18.10.5, “Handling a file upload request from programmatic clients”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-multipart-forms-non-browsers)and[Section18.10, “Spring’s multipart \(file upload\) support”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-multipart).

* `@SessionAttribute`annotated parameters for access to existing, permanent session attributes \(e.g. user authentication object\) as opposed to model attributes temporarily stored in the session as part of a controller workflow via`@SessionAttributes`.

* `@RequestAttribute`annotated parameters for access to request attributes.

* `HttpEntity`parameters for access to the Servlet request HTTP headers and contents. The request stream will be converted to the entity body using`HttpMessageConverter`s. See[the section called “Using HttpEntity”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-ann-httpentity).

* `java.util.Map`/`org.springframework.ui.Model`/`org.springframework.ui.ModelMap`for enriching the implicit model that is exposed to the web view.

* `org.springframework.web.servlet.mvc.support.RedirectAttributes`to specify the exact set of attributes to use in case of a redirect and also to add flash attributes \(attributes stored temporarily on the server-side to make them available to the request after the redirect\). See[the section called “Passing Data To the Redirect Target”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-redirecting-passing-data)and[Section18.6, “Using flash attributes”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-flash-attributes).

* Command or form objects to bind request parameters to bean properties \(via setters\) or directly to fields, with customizable type conversion, depending on`@InitBinder`methods and/or the HandlerAdapter configuration. See the`webBindingInitializer`property on`RequestMappingHandlerAdapter`. Such command objects along with their validation results will be exposed as model attributes by default, using the command class name - e.g. model attribute "orderAddress" for a command object of type "some.package.OrderAddress". The`ModelAttribute`annotation can be used on a method argument to customize the model attribute name used.

* `org.springframework.validation.Errors`/`org.springframework.validation.BindingResult`validation results for a preceding command or form object \(the immediately preceding method argument\).

* `org.springframework.web.bind.support.SessionStatus`status handle for marking form processing as complete, which triggers the cleanup of session attributes that have been indicated by the`@SessionAttributes`annotation at the handler type level.

* `org.springframework.web.util.UriComponentsBuilder`a builder for preparing a URL relative to the current request’s host, port, scheme, context path, and the literal part of the servlet mapping.

The`Errors`or`BindingResult`parameters have to follow the model object that is being bound immediately as the method signature might have more than one model object and Spring will create a separate`BindingResult`instance for each of them so the following sample won’t work:

**Invalid ordering of BindingResult and @ModelAttribute.**

```java
@PostMapping
public String processSubmit(@ModelAttribute("pet") Pet pet, Model model, BindingResult result) { ... }
```

Note, that there is a`Model`parameter in between`Pet`and`BindingResult`. To get this working you have to reorder the parameters as follows:

```java
@PostMapping
public String processSubmit(@ModelAttribute("pet") Pet pet, BindingResult result, Model model) { ... }
```

| ![](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png) |
| :--- |
| JDK 1.8’s`java.util.Optional`is supported as a method parameter type with annotations that have a`required`attribute \(e.g.`@RequestParam`,`@RequestHeader`, etc\). The use of`java.util.Optional`in those cases is equivalent to having`required=false`. |

#### Supported method return types

The following are the supported return types:

* A`ModelAndView`object, with the model implicitly enriched with command objects and the results of`@ModelAttribute`annotated reference data accessor methods.

* A`Model`object, with the view name implicitly determined through a`RequestToViewNameTranslator`and the model implicitly enriched with command objects and the results of`@ModelAttribute`annotated reference data accessor methods.

* A`Map`object for exposing a model, with the view name implicitly determined through a`RequestToViewNameTranslator`and the model implicitly enriched with command objects and the results of`@ModelAttribute`annotated reference data accessor methods.

* A`View`object, with the model implicitly determined through command objects and`@ModelAttribute`annotated reference data accessor methods. The handler method may also programmatically enrich the model by declaring a`Model`argument \(see above\).

* A`String`value that is interpreted as the logical view name, with the model implicitly determined through command objects and`@ModelAttribute`annotated reference data accessor methods. The handler method may also programmatically enrich the model by declaring a`Model`argument \(see above\).

* `void`if the method handles the response itself \(by writing the response content directly, declaring an argument of type`ServletResponse`/`HttpServletResponse`for that purpose\) or if the view name is supposed to be implicitly determined through a`RequestToViewNameTranslator`\(not declaring a response argument in the handler method signature\).

* If the method is annotated with`@ResponseBody`, the return type is written to the response HTTP body. The return value will be converted to the declared method argument type using`HttpMessageConverter`s. See[the section called “Mapping the response body with the @ResponseBody annotation”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-ann-responsebody).

* An`HttpEntity`or`ResponseEntity`object to provide access to the Servlet response HTTP headers and contents. The entity body will be converted to the response stream using`HttpMessageConverter`s. See[the section called “Using HttpEntity”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-ann-httpentity).

* An`HttpHeaders`object to return a response with no body.

* A`Callable`can be returned when the application wants to produce the return value asynchronously in a thread managed by Spring MVC.

* A`DeferredResult`can be returned when the application wants to produce the return value from a thread of its own choosing.

* A`ListenableFuture`or`CompletableFuture`/`CompletionStage`can be returned when the application wants to produce the value from a thread pool submission.

* A`ResponseBodyEmitter`can be returned to write multiple objects to the response asynchronously; also supported as the body within a`ResponseEntity`.

* An`SseEmitter`can be returned to write Server-Sent Events to the response asynchronously; also supported as the body within a`ResponseEntity`.

* A`StreamingResponseBody`can be returned to write to the response OutputStream asynchronously; also supported as the body within a`ResponseEntity`.

* Any other return type is considered to be a single model attribute to be exposed to the view, using the attribute name specified through`@ModelAttribute`at the method level \(or the default attribute name based on the return type class name\). The model is implicitly enriched with command objects and the results of`@ModelAttribute`annotated reference data accessor methods.

#### Binding request parameters to method parameters with @RequestParam

Use the`@RequestParam`annotation to bind request parameters to a method parameter in your controller.

The following code snippet shows the usage:

```java
@Controller
@RequestMapping("/pets")
@SessionAttributes("pet")
public class EditPetForm {

	// ...

	@GetMapping
	public String setupForm(@RequestParam("petId") int petId, ModelMap model) {
		Pet pet = this.clinic.loadPet(petId);
		model.addAttribute("pet", pet);
		return "petForm";
	}

	// ...

}
```

Parameters using this annotation are required by default, but you can specify that a parameter is optional by setting`@RequestParam`'s`required`attribute to`false`\(e.g.,`@RequestParam(name="id", required=false)`\).

Type conversion is applied automatically if the target method parameter type is not`String`. See[the section called “Method Parameters And Type Conversion”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-ann-typeconversion).

When an`@RequestParam`annotation is used on a`Map`or`MultiValueMap`argument, the map is populated with all request parameters.

#### Mapping the request body with the @RequestBody annotation

The`@RequestBody`method parameter annotation indicates that a method parameter should be bound to the value of the HTTP request body. For example:

```java
@PutMapping("/something")
public void handle(@RequestBody String body, Writer writer) throws IOException {
	writer.write(body);
}
```

You convert the request body to the method argument by using an`HttpMessageConverter`.`HttpMessageConverter`is responsible for converting from the HTTP request message to an object and converting from an object to the HTTP response body. The`RequestMappingHandlerAdapter`supports the`@RequestBody`annotation with the following default`HttpMessageConverters`:

* `ByteArrayHttpMessageConverter`converts byte arrays.

* `StringHttpMessageConverter`converts strings.

* `FormHttpMessageConverter`converts form data to/from a MultiValueMap&lt;String, String&gt;.

* `SourceHttpMessageConverter`converts to/from a javax.xml.transform.Source.

For more information on these converters, see[Message Converters](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/remoting.html#rest-message-conversion). Also note that if using the MVC namespace or the MVC Java config, a wider range of message converters are registered by default. See[Section18.16.1, “Enabling the MVC Java Config or the MVC XML Namespace”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-config-enable)for more information.

If you intend to read and write XML, you will need to configure the`MarshallingHttpMessageConverter`with a specific`Marshaller`and an`Unmarshaller`implementation from the`org.springframework.oxm`package. The example below shows how to do that directly in your configuration but if your application is configured through the MVC namespace or the MVC Java config see[Section18.16.1, “Enabling the MVC Java Config or the MVC XML Namespace”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-config-enable)instead.

```java
<bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter">
	<property name="messageConverters">
		<util:list id="beanList">
			<ref bean="stringHttpMessageConverter"/>
			<ref bean="marshallingHttpMessageConverter"/>
		</util:list>
	</property>
</bean>

<bean id="stringHttpMessageConverter"
		class="org.springframework.http.converter.StringHttpMessageConverter"/>

<bean id="marshallingHttpMessageConverter"
		class="org.springframework.http.converter.xml.MarshallingHttpMessageConverter">
	<property name="marshaller" ref="castorMarshaller"/>
	<property name="unmarshaller" ref="castorMarshaller"/>
</bean>
```

An`@RequestBody`method parameter can be annotated with`@Valid`, in which case it will be validated using the configured`Validator`instance. When using the MVC namespace or the MVC Java config, a JSR-303 validator is configured automatically assuming a JSR-303 implementation is available on the classpath.

Just like with`@ModelAttribute`parameters, an`Errors`argument can be used to examine the errors. If such an argument is not declared, a`MethodArgumentNotValidException`will be raised. The exception is handled in the`DefaultHandlerExceptionResolver`, which sends a`400`error back to the client.

| ![](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png) |
| :--- |
| Also see[Section18.16.1, “Enabling the MVC Java Config or the MVC XML Namespace”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-config-enable)for information on configuring message converters and a validator through the MVC namespace or the MVC Java config. |

#### Mapping the response body with the @ResponseBody annotation

The`@ResponseBody`annotation is similar to`@RequestBody`. This annotation can be placed on a method and indicates that the return type should be written straight to the HTTP response body \(and not placed in a Model, or interpreted as a view name\). For example:

```java
@GetMapping("/something")
@ResponseBody
public String helloWorld() {
	return "Hello World";
}
```

The above example will result in the text`Hello World`being written to the HTTP response stream.

As with`@RequestBody`, Spring converts the returned object to a response body by using an`HttpMessageConverter`. For more information on these converters, see the previous section and[Message Converters](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/remoting.html#rest-message-conversion).

#### Creating REST Controllers with the @RestController annotation

It’s a very common use case to have Controllers implement a REST API, thus serving only JSON, XML or custom MediaType content. For convenience, instead of annotating all your`@RequestMapping`methods with`@ResponseBody`, you can annotate your controller Class with`@RestController`.

[`@RestController`](http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/web/bind/annotation/RestController.html)is a stereotype annotation that combines`@ResponseBody`and`@Controller`. More than that, it gives more meaning to your Controller and also may carry additional semantics in future releases of the framework.

As with regular`@Controller`s, a`@RestController`may be assisted by`@ControllerAdvice`or`@RestControllerAdvice`beans. See the[the section called “Advising controllers with @ControllerAdvice and @RestControllerAdvice”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-ann-controller-advice)section for more details.

#### Using HttpEntity

The`HttpEntity`is similar to`@RequestBody`and`@ResponseBody`. Besides getting access to the request and response body,`HttpEntity`\(and the response-specific subclass`ResponseEntity`\) also allows access to the request and response headers, like so:

```java
@RequestMapping("/something")
public ResponseEntity<String> handle(HttpEntity<byte[]> requestEntity) throws UnsupportedEncodingException {
	String requestHeader = requestEntity.getHeaders().getFirst("MyRequestHeader");
	byte[] requestBody = requestEntity.getBody();

	// do something with request header and body

	HttpHeaders responseHeaders = new HttpHeaders();
	responseHeaders.set("MyResponseHeader", "MyValue");
	return new ResponseEntity<String>("Hello World", responseHeaders, HttpStatus.CREATED);
}
```

The above example gets the value of the`MyRequestHeader`request header, and reads the body as a byte array. It adds the`MyResponseHeader`to the response, writes`Hello World`to the response stream, and sets the response status code to 201 \(Created\).

As with`@RequestBody`and`@ResponseBody`, Spring uses`HttpMessageConverter`to convert from and to the request and response streams. For more information on these converters, see the previous section and[Message Converters](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/remoting.html#rest-message-conversion).

####  Using @ModelAttribute on a method

The`@ModelAttribute`annotation can be used on methods or on method arguments. This section explains its usage on methods while the next section explains its usage on method arguments.

An`@ModelAttribute`on a method indicates the purpose of that method is to add one or more model attributes. Such methods support the same argument types as`@RequestMapping`methods but cannot be mapped directly to requests. Instead`@ModelAttribute`methods in a controller are invoked before`@RequestMapping`methods, within the same controller. A couple of examples:

```java
// Add one attribute
// The return value of the method is added to the model under the name "account"
// You can customize the name via @ModelAttribute("myAccount")

@ModelAttribute
public Account addAccount(@RequestParam String number) {
	return accountManager.findAccount(number);
}

// Add multiple attributes

@ModelAttribute
public void populateModel(@RequestParam String number, Model model) {
	model.addAttribute(accountManager.findAccount(number));
	// add more ...
}
```

`@ModelAttribute`methods are used to populate the model with commonly needed attributes for example to fill a drop-down with states or with pet types, or to retrieve a command object like Account in order to use it to represent the data on an HTML form. The latter case is further discussed in the next section.

Note the two styles of`@ModelAttribute`methods. In the first, the method adds an attribute implicitly by returning it. In the second, the method accepts a`Model`and adds any number of model attributes to it. You can choose between the two styles depending on your needs.

A controller can have any number of`@ModelAttribute`methods. All such methods are invoked before`@RequestMapping`methods of the same controller.

`@ModelAttribute`methods can also be defined in an`@ControllerAdvice`-annotated class and such methods apply to many controllers. See the[the section called “Advising controllers with @ControllerAdvice and @RestControllerAdvice”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-ann-controller-advice)section for more details.

| ![](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/tip.png) |
| :--- |
| What happens when a model attribute name is not explicitly specified? In such cases a default name is assigned to the model attribute based on its type. For example if the method returns an object of type`Account`, the default name used is "account". You can change that through the value of the`@ModelAttribute`annotation. If adding attributes directly to the`Model`, use the appropriate overloaded`addAttribute(..)`method - i.e., with or without an attribute name. |

The`@ModelAttribute`annotation can be used on`@RequestMapping`methods as well. In that case the return value of the`@RequestMapping`method is interpreted as a model attribute rather than as a view name. The view name is then derived based on view name conventions instead, much like for methods returning`void` — see[Section18.13.3, “The View - RequestToViewNameTranslator”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-coc-r2vnt).

#### Using @ModelAttribute on a method argument

As explained in the previous section`@ModelAttribute`can be used on methods or on method arguments. This section explains its usage on method arguments.

An`@ModelAttribute`on a method argument indicates the argument should be retrieved from the model. If not present in the model, the argument should be instantiated first and then added to the model. Once present in the model, the argument’s fields should be populated from all request parameters that have matching names. This is known as data binding in Spring MVC, a very useful mechanism that saves you from having to parse each form field individually.

```java
@PostMapping("/owners/{ownerId}/pets/{petId}/edit")
public String processSubmit(@ModelAttribute Pet pet) { }
```

Given the above example where can the Pet instance come from? There are several options:

* It may already be in the model due to use of`@SessionAttributes` — see[the section called “Using @SessionAttributes to store model attributes in the HTTP session between requests”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-ann-sessionattrib).

* It may already be in the model due to an`@ModelAttribute`method in the same controller — as explained in the previous section.

* It may be retrieved based on a URI template variable and type converter \(explained in more detail below\).

* It may be instantiated using its default constructor.

An`@ModelAttribute`method is a common way to retrieve an attribute from the database, which may optionally be stored between requests through the use of`@SessionAttributes`. In some cases it may be convenient to retrieve the attribute by using an URI template variable and a type converter. Here is an example:

```java
@PutMapping("/accounts/{account}")
public String save(@ModelAttribute("account") Account account) {
	// ...
}
```

In this example the name of the model attribute \(i.e. "account"\) matches the name of a URI template variable. If you register`Converter`that can turn the`String`account value into an`Account`instance, then the above example will work without the need for an`@ModelAttribute`method.

The next step is data binding. The`WebDataBinder`class matches request parameter names — including query string parameters and form fields — to model attribute fields by name. Matching fields are populated after type conversion \(from String to the target field type\) has been applied where necessary. Data binding and validation are covered in[Chapter5,_Validation, Data Binding, and Type Conversion_](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/validation.html). Customizing the data binding process for a controller level is covered in[the section called “Customizing WebDataBinder initialization”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-ann-webdatabinder).

As a result of data binding there may be errors such as missing required fields or type conversion errors. To check for such errors add a`BindingResult`argument immediately following the`@ModelAttribute`argument:

```java
@PostMapping("/owners/{ownerId}/pets/{petId}/edit")
public String processSubmit(@ModelAttribute("pet") Pet pet, BindingResult result) {

	if (result.hasErrors()) {
		return "petForm";
	}

	// ...

}
```

With a`BindingResult`you can check if errors were found in which case it’s common to render the same form where the errors can be shown with the help of Spring’s\`\`form tag.

Note that in some cases it may be useful to gain access to an attribute in the model without data binding. For such cases you may inject the`Model`into the controller or alternatively use the`binding`flag on the annotation:

```java
@ModelAttribute
public AccountForm setUpForm() {
    return new AccountForm();
}

@ModelAttribute
public Account findAccount(@PathVariable String accountId) {
    return accountRepository.findOne(accountId);
}

@PostMapping("update")
public String update(@Valid AccountUpdateForm form, BindingResult result,
        @ModelAttribute(binding=false) Account account) {

    // ...
}
```

In addition to data binding you can also invoke validation using your own custom validator passing the same`BindingResult`that was used to record data binding errors. That allows for data binding and validation errors to be accumulated in one place and subsequently reported back to the user:

```java
@PostMapping("/owners/{ownerId}/pets/{petId}/edit")
public String processSubmit(@ModelAttribute("pet") Pet pet, BindingResult result) {

	new PetValidator().validate(pet, result);
	if (result.hasErrors()) {
		return "petForm";
	}

	// ...

}
```

Or you can have validation invoked automatically by adding the JSR-303`@Valid`annotation:

```java
@PostMapping("/owners/{ownerId}/pets/{petId}/edit")
public String processSubmit(@Valid @ModelAttribute("pet") Pet pet, BindingResult result) {

	if (result.hasErrors()) {
		return "petForm";
	}

	// ...

}
```

See[Section5.8, “Spring Validation”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/validation.html#validation-beanvalidation)and[Chapter5,_Validation, Data Binding, and Type Conversion_](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/validation.html)for details on how to configure and use validation.

#### Using @SessionAttributes to store model attributes in the HTTP session between requests

The type-level`@SessionAttributes`annotation declares session attributes used by a specific handler. This will typically list the names of model attributes or types of model attributes which should be transparently stored in the session or some conversational storage, serving as form-backing beans between subsequent requests.

The following code snippet shows the usage of this annotation, specifying the model attribute name:

```java
@Controller
@RequestMapping("/editPet.do")
@SessionAttributes("pet")
public class EditPetForm {
	// ...
}
```

#### Using @SessionAttribute to access pre-existing global session attributes

If you need access to pre-existing session attributes that are managed globally, i.e. outside the controller \(e.g. by a filter\), and may or may not be present use the`@SessionAttribute`annotation on a method parameter:

```java
@RequestMapping("/")
public String handle(@SessionAttribute User user) {
	// ...
}
```

For use cases that require adding or removing session attributes consider injecting`org.springframework.web.context.request.WebRequest`or`javax.servlet.http.HttpSession`into the controller method.

For temporary storage of model attributes in the session as part of a controller workflow consider using`SessionAttributes`as described in[the section called “Using @SessionAttributes to store model attributes in the HTTP session between requests”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-ann-sessionattrib).

#### Using @RequestAttribute to access request attributes

Similar to`@SessionAttribute`the`@RequestAttribute`annotation can be used to access pre-existing request attributes created by a filter or interceptor:

```java
@RequestMapping("/")
public String handle(@RequestAttribute Client client) {
	// ...
}
```

#### Working with "application/x-www-form-urlencoded" data

The previous sections covered use of`@ModelAttribute`to support form submission requests from browser clients. The same annotation is recommended for use with requests from non-browser clients as well. However there is one notable difference when it comes to working with HTTP PUT requests. Browsers can submit form data via HTTP GET or HTTP POST. Non-browser clients can also submit forms via HTTP PUT. This presents a challenge because the Servlet specification requires the`ServletRequest.getParameter*()`family of methods to support form field access only for HTTP POST, not for HTTP PUT.

To support HTTP PUT and PATCH requests, the`spring-web`module provides the filter`HttpPutFormContentFilter`, which can be configured in`web.xml`:

```java
<filter>
	<filter-name>httpPutFormFilter</filter-name>
	<filter-class>org.springframework.web.filter.HttpPutFormContentFilter</filter-class>
</filter>

<filter-mapping>
	<filter-name>httpPutFormFilter</filter-name>
	<servlet-name>dispatcherServlet</servlet-name>
</filter-mapping>

<servlet>
	<servlet-name>dispatcherServlet</servlet-name>
	<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
</servlet>
```

The above filter intercepts HTTP PUT and PATCH requests with content type`application/x-www-form-urlencoded`, reads the form data from the body of the request, and wraps the`ServletRequest`in order to make the form data available through the`ServletRequest.getParameter*()`family of methods.

| ![](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png) |
| :--- |
| As`HttpPutFormContentFilter`consumes the body of the request, it should not be configured for PUT or PATCH URLs that rely on other converters for`application/x-www-form-urlencoded`. This includes`@RequestBody MultiValueMap`and`HttpEntity>`. |

#### Mapping cookie values with the @CookieValue annotation

The`@CookieValue`annotation allows a method parameter to be bound to the value of an HTTP cookie.

Let us consider that the following cookie has been received with an http request:

```java
JSESSIONID=415A4AC178C59DACE0B2C9CA727CDD84
```

The following code sample demonstrates how to get the value of the`JSESSIONID`cookie:

```java
@RequestMapping("/displayHeaderInfo.do")
public void displayHeaderInfo(@CookieValue("JSESSIONID") String cookie) {
	//...
}
```

Type conversion is applied automatically if the target method parameter type is not`String`. See[the section called “Method Parameters And Type Conversion”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-ann-typeconversion).

#### Mapping request header attributes with the @RequestHeader annotation

The`@RequestHeader`annotation allows a method parameter to be bound to a request header.

Here is a sample request header:

```java
Host                    localhost:8080
Accept                  text/html,application/xhtml+xml,application/xml;q=0.9
Accept-Language         fr,en-gb;q=0.7,en;q=0.3
Accept-Encoding         gzip,deflate
Accept-Charset          ISO-8859-1,utf-8;q=0.7,*;q=0.7
Keep-Alive              300
```

The following code sample demonstrates how to get the value of the`Accept-Encoding`and`Keep-Alive`headers:

```java
@RequestMapping("/displayHeaderInfo.do")
public void displayHeaderInfo(@RequestHeader("Accept-Encoding") String encoding,
		@RequestHeader("Keep-Alive") long keepAlive) {
	//...
}
```

Type conversion is applied automatically if the method parameter is not`String`. See[the section called “Method Parameters And Type Conversion”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-ann-typeconversion).

When an`@RequestHeader`annotation is used on a`Map`,`MultiValueMap`, or`HttpHeaders`argument, the map is populated with all header values.

| ![](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/tip.png) |
| :--- |
| Built-in support is available for converting a comma-separated string into an array/collection of strings or other types known to the type conversion system. For example a method parameter annotated with`@RequestHeader("Accept")`may be of type`String`but also`String[]`or`List`. |

#### Method Parameters And Type Conversion

String-based values extracted from the request including request parameters, path variables, request headers, and cookie values may need to be converted to the target type of the method parameter or field \(e.g., binding a request parameter to a field in an`@ModelAttribute`parameter\) they’re bound to. If the target type is not`String`, Spring automatically converts to the appropriate type. All simple types such as int, long, Date, etc. are supported. You can further customize the conversion process through a`WebDataBinder`\(see[the section called “Customizing WebDataBinder initialization”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-ann-webdatabinder)\) or by registering`Formatters`with the`FormattingConversionService`\(see[Section 5.6, “Spring Field Formatting”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/validation.html#format)\).

#### Customizing WebDataBinder initialization

To customize request parameter binding with PropertyEditors through Spring’s`WebDataBinder`, you can use`@InitBinder`-annotated methods within your controller,`@InitBinder`methods within an`@ControllerAdvice`class, or provide a custom`WebBindingInitializer`. See the[the section called “Advising controllers with @ControllerAdvice and @RestControllerAdvice”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-ann-controller-advice)section for more details.

##### Customizing data binding with @InitBinder

Annotating controller methods with`@InitBinder`allows you to configure web data binding directly within your controller class.`@InitBinder`identifies methods that initialize the`WebDataBinder`that will be used to populate command and form object arguments of annotated handler methods.

Such init-binder methods support all arguments that`@RequestMapping`methods support, except for command/form objects and corresponding validation result objects. Init-binder methods must not have a return value. Thus, they are usually declared as`void`. Typical arguments include`WebDataBinder`in combination with`WebRequest`or`java.util.Locale`, allowing code to register context-specific editors.

The following example demonstrates the use of`@InitBinder`to configure a`CustomDateEditor`for all`java.util.Date`form properties.

```java
@Controller
public class MyFormController {

	@InitBinder
	protected void initBinder(WebDataBinder binder) {
		SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
		dateFormat.setLenient(false);
		binder.registerCustomEditor(Date.class, new CustomDateEditor(dateFormat, false));
	}

	// ...
}
```

Alternatively, as of Spring 4.2, consider using`addCustomFormatter`to specify`Formatter`implementations instead of`PropertyEditor`instances. This is particularly useful if you happen to have a`Formatter`-based setup in a shared`FormattingConversionService`as well, with the same approach to be reused for controller-specific tweaking of the binding rules.

```java
@Controller
public class MyFormController {

	@InitBinder
	protected void initBinder(WebDataBinder binder) {
		binder.addCustomFormatter(new DateFormatter("yyyy-MM-dd"));
	}

	// ...
}
```

##### Configuring a custom WebBindingInitializer

To externalize data binding initialization, you can provide a custom implementation of the`WebBindingInitializer`interface, which you then enable by supplying a custom bean configuration for an`RequestMappingHandlerAdapter`, thus overriding the default configuration.

The following example from the PetClinic application shows a configuration using a custom implementation of the`WebBindingInitializer`interface,`org.springframework.samples.petclinic.web.ClinicBindingInitializer`, which configures PropertyEditors required by several of the PetClinic controllers.

```java
<bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter">
	<property name="cacheSeconds" value="0"/>
	<property name="webBindingInitializer">
		<bean class="org.springframework.samples.petclinic.web.ClinicBindingInitializer"/>
	</property>
</bean>
```

`@InitBinder`methods can also be defined in an`@ControllerAdvice`-annotated class in which case they apply to matching controllers. This provides an alternative to using a`WebBindingInitializer`. See the[the section called “Advising controllers with @ControllerAdvice and @RestControllerAdvice”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-ann-controller-advice)section for more details.

#### Advising controllers with @ControllerAdvice and @RestControllerAdvice

The`@ControllerAdvice`annotation is a component annotation allowing implementation classes to be auto-detected through classpath scanning. It is automatically enabled when using the MVC namespace or the MVC Java config.

Classes annotated with`@ControllerAdvice`can contain`@ExceptionHandler`,`@InitBinder`, and`@ModelAttribute`annotated methods, and these methods will apply to`@RequestMapping`methods across all controller hierarchies as opposed to the controller hierarchy within which they are declared.

`@RestControllerAdvice`is an alternative where`@ExceptionHandler`methods assume`@ResponseBody`semantics by default.

Both`@ControllerAdvice`and`@RestControllerAdvice`can target a subset of controllers:

```java
// Target all Controllers annotated with @RestController
@ControllerAdvice(annotations = RestController.class)
public class AnnotationAdvice {}

// Target all Controllers within specific packages
@ControllerAdvice("org.example.controllers")
public class BasePackageAdvice {}

// Target all Controllers assignable to specific classes
@ControllerAdvice(assignableTypes = {ControllerInterface.class, AbstractController.class})
public class AssignableTypesAdvice {}
```

Check out the[`@ControllerAdvice`documentation](http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/web/bind/annotation/ControllerAdvice.html)for more details.

#### Jackson Serialization View Support

It can sometimes be useful to filter contextually the object that will be serialized to the HTTP response body. In order to provide such capability, Spring MVC has built-in support for rendering with[Jackson’s Serialization Views](http://wiki.fasterxml.com/JacksonJsonViews).

To use it with an`@ResponseBody`controller method or controller methods that return`ResponseEntity`, simply add the`@JsonView`annotation with a class argument specifying the view class or interface to be used:

```java
@RestController
public class UserController {

	@GetMapping("/user")
	@JsonView(User.WithoutPasswordView.class)
	public User getUser() {
		return new User("eric", "7!jd#h23");
	}
}

public class User {

	public interface WithoutPasswordView {};
	public interface WithPasswordView extends WithoutPasswordView {};

	private String username;
	private String password;

	public User() {
	}

	public User(String username, String password) {
		this.username = username;
		this.password = password;
	}

	@JsonView(WithoutPasswordView.class)
	public String getUsername() {
		return this.username;
	}

	@JsonView(WithPasswordView.class)
	public String getPassword() {
		return this.password;
	}
}
```

| ![](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png) |
| :--- |
| Note that despite`@JsonView`allowing for more than one class to be specified, the use on a controller method is only supported with exactly one class argument. Consider the use of a composite interface if you need to enable multiple views. |

For controllers relying on view resolution, simply add the serialization view class to the model:

```java
@Controller
public class UserController extends AbstractController {

	@GetMapping("/user")
	public String getUser(Model model) {
		model.addAttribute("user", new User("eric", "7!jd#h23"));
		model.addAttribute(JsonView.class.getName(), User.WithoutPasswordView.class);
		return "userView";
	}
}
```

#### Jackson JSONP Support

In order to enable[JSONP](https://en.wikipedia.org/wiki/JSONP)support for`@ResponseBody`and`ResponseEntity`methods, declare an`@ControllerAdvice`bean that extends`AbstractJsonpResponseBodyAdvice`as shown below where the constructor argument indicates the JSONP query parameter name\(s\):

```java
@ControllerAdvice
public class JsonpAdvice extends AbstractJsonpResponseBodyAdvice {

	public JsonpAdvice() {
		super("callback");
	}
}
```

For controllers relying on view resolution, JSONP is automatically enabled when the request has a query parameter named`jsonp`or`callback`. Those names can be customized through`jsonpParameterNames`property.

