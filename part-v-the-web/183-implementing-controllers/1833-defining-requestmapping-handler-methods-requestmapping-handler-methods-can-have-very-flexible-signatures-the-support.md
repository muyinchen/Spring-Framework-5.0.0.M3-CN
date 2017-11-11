### 18.3.3 定义@RequestMapping 处理方法

`@RequestMapping`处理方法可以有非常灵活的签名。 支持的方法参数和返回值将在以下部分中介绍。 大多数参数可以按任意顺序使用，唯一的例外是`BindingResult`参数。 这将在下一节中介绍。

| ![](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png) |
| :--- |
| Spring 3.1为`@RequestMapping`方法引入了一组新的支持类，分别称为`RequestMappingHandlerMapping`和`RequestMappingHandlerAdapter`。 它们被推荐使用，甚至需要利用Spring MVC 3.1中的新特性并继续前进。 新的支持类默认从MVC命名空间启用，并且使用MVC Java配置，但是如果两者都不使用，则必须明确配置。 |

#### 支持的方法参数类型

下面是支持的方法参数类型：

* `org.springframework.web.context.request.WebRequest`或 `org.springframework.web.context.request.NativeWebRequest`允许通用请求参数访问以及请求/会话属性访问，而不涉及本机Servlet API。

* Request 或 response 对象\(Servlet API\). 选择任意特定的请求或响应类型, 例如  
  `ServletRequest`或`HttpServletRequest`或 Spring’s`MultipartRequest`/`MultipartHttpServletRequest`  
  .

* Session对象（Servlet API）类型为`HttpSession`。 此类型的参数强制存在相应的会话。 因此，这样的论证从不为`null`。

| ![](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png) |
| :--- |
| 会话访问可能不是线程安全的，特别是在Servlet环境中。 如果允许多个请求同时访问会话，请考虑将`RequestMappingHandlerAdapter`的“synchronizeOnSession”标志设置为“true”。. |

* `java.servlet.http.PushBuilder`用于关联的Servlet 4.0推送构建器API，允许编程的HTTP / 2资源推送。

* `java.security.Principal`（或一个特定的`Principal`实现类（如果已知）），包含当前验证的用户。

* `org.springframework.http.HttpMethod`为HTTP请求方法，表示为Spring的`HttpMethod`枚举。

* 由当前请求区域设置的`java.util.Locale`，由最具体的语言环境解析器确定，实际上是在MVC环境中配置的`LocaleResolver`/ `LocaleContextResolver`。

* 与当前请求相关联的时区的`java.util.TimeZone`（Java 6+）/ `java.time.ZoneId`（Java 8+），由`LocaleContextResolver`确定。

* `java.io.InputStream` / `java.io.Reader`，用于访问请求的内容。该值是由Servlet API公开的原始InputStream / Reader。

* `java.io.OutputStream` / `java.io.Writer`用于生成响应的内容。该值是由Servlet API公开的原始OutputStream / Writer。

* `@PathVariable`注释参数，用于访问URI模板变量。请参阅[the section called “URI Template Patterns”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-ann-requestmapping-uri-templates)  
  .

* `@MatrixVariable`注释参数，用于访问位于URI路径段中的名称/值对。请参阅 [the section called “Matrix Variables”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-ann-matrix-variables)。

* `@RequestParam`用于访问特定Servlet请求参数的注释参数。参数值将转换为声明的方法参数类型。请参阅[the section called “Binding request parameters to method parameters with @RequestParam”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-ann-requestparam)。

* `@RequestHeader`用于访问特定Servlet请求HTTP标头的注释参数。参数值将转换为声明的方法参数类型。请参阅 [the section called “Mapping request header attributes with the @RequestHeader annotation”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-ann-requestheader)。

* `@RequestBody`用于访问HTTP请求体的注释参数。使用`HttpMessageConverters`将参数值转换为声明的方法参数类型。请参阅[the section called “Mapping the request body with the @RequestBody annotation”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-ann-requestbody)。

* @RequestPart注释参数，用于访问“multipart / form-data”请求部分的内容。请参见[Section 18.10.5, “Handling a file upload request from programmatic clients”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-multipart-forms-non-browsers)和[Section 18.10, “Spring’s multipart \(file upload\) support”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-multipart)

* `@SessionAttribute`用于访问现有的永久会话属性（例如，用户认证对象）的注释参数，而不是通过`@SessionAttributes`作为控制器工作流的一部分临时存储在会话中的模型属性。

* `@RequestAttribute`用于访问请求属性的注释参数。

* `HttpEntity`参数访问Servlet请求HTTP头和内容。请求流将使用`HttpMessageConverters`转换为实体。请参阅 [the section called “Using HttpEntity”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-ann-httpentity)。

* `java.util.Map` / `org.springframework.ui.Model` / `org.springframework.ui.ModelMap`用于丰富暴露于Web视图的隐式模型。

* `org.springframework.web.servlet.mvc.support.RedirectAttributes`来指定在重定向情况下使用的精确的属性集，并且还添加Flash属性（临时存储在服务器端的属性，使其可以在请求之后使用重定向）。请参见   
  [the section called “Passing Data To the Redirect Target”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-redirecting-passing-data)和[Section 18.6, “Using flash attributes”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-flash-attributes)。

* 根据`@InitBinder`方法和/或HandlerAdapter配置，命令或表单对象将请求参数绑定到bean属性（通过setter）或直接转换为字段，并进行可定制的类型转换。请参阅`RequestMappingHandlerAdapter`上的`webBindingInitializer`属性。默认情况下，这些命令对象及其验证结果将作为模型属性公开，使用命令类名称 – 例如。对于“some.package.OrderAddress”类型的命令对象的model属性“orderAddress”。 `ModelAttribute`注释可以用于方法参数来自定义所使用的模型属性名称。

* `org.springframework.validation.Errors` / `org.springframework.validation.BindingResult`验证前一个命令或表单对象的结果（即在前面的方法参数）。

* 用于将表单处理标记为完整的`org.springframework.web.bind.support.SessionStatus`状态句柄，它触发在处理程序类型级别上由`@SessionAttributes`注释指示的会话属性的清除。

* `org.springframework.web.util.UriComponentsBuilder`用于准备与当前请求的主机，端口，方案，上下文路径以及servlet映射的文字部分相关的URL的构建器。

The`Errors`or`BindingResult`parameters have to follow the model object that is being bound immediately as the method signature might have more than one model object and Spring will create a separate`BindingResult`instance for each of them so the following sample won’t work:

`Errors`或`BindingResult`参数必须遵循正在绑定的模型对象，因为方法签名可能有多个模型对象，Spring将为每个模型对象创建一个单独的`BindingResult`实例，因此以下示例将不起作用：

**BindingResult和@ModelAttribute的排序无效。**

```java
@PostMapping
public String processSubmit(@ModelAttribute("pet") Pet pet, Model model, BindingResult result) { ... }
```

注意，`Pet`和`BindingResult`之间有一个`Model`参数。 要使其工作，您必须重新排序参数如下：

```java
@PostMapping
public String processSubmit(@ModelAttribute("pet") Pet pet, BindingResult result, Model model) { ... }
```

| ![](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png) |
| :--- |
| 支持JDK 1.8的`java.util.Optional`作为方法参数类型，其注解具有所需的属性（例如`@RequestParam`，`@RequestHeader`等）。 在这些情况下使用`java.util.Optional`相当于`required=false`。 |

#### 支持的方法返回类型

以下是支持的返回类型：

* 一个`ModelAndView`对象，其中模型隐含地丰富了命令对象和`@ModelAttribute`注解引用数据访问器方法的结果。

* 一个`Model`对象，其视图名称通过`RequestToViewNameTranslator`隐式确定，隐式丰富了命令对象的模型以及`@ModelAttribute`注解引用数据访问器方法的结果。

* 用于暴露模型的`Map`对象，其视图名称通过`RequestToViewNameTranslator`隐式确定，隐式丰富了命令对象的模型以及`@ModelAttribute`注解引用数据访问器方法的结果。

* 一个`View`对象，其模型通过命令对象和`@ModelAttribute`注解引用数据访问器方法隐式确定。处理程序方法也可以通过声明一个`Model`参数（见上文）以编程方式丰富模型。

* 解释为逻辑视图名称的`String`值，模型通过命令对象和`@ModelAttribute`注解引用数据访问器方法隐式确定。处理程序方法也可以通过声明一个`Model`参数（见上文）以编程方式丰富模型。

* 如果方法处理响应本身（通过直接写入响应内容，为此目的声明一个类型为`ServletResponse` / `HttpServletResponse`的参数），或者如果视图名称通过`RequestToViewNameTranslator`隐式确定（不在处理程序方法签名）。

* 如果该方法用`@ResponseBody`注解，则返回类型将写入响应HTTP主体。返回值将使用`HttpMessageConverters`转换为声明的方法参数类型。请参阅 [the section called “Mapping the response body with the @ResponseBody annotation”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-ann-responsebody)。

* 一个`HttpEntity `或`ResponseEntity`对象来提供对Servlet响应HTTP头和内容的访问。实体将使用`HttpMessageConverters`转换为响应流。请参阅[the section called “Using HttpEntity”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-ann-httpentity).

* 一个`HttpHeaders`对象返回没有正文的响应。

* 当应用程序想要在由Spring MVC管理的线程中异步生成返回值时，可以返回`Callable`。

* 当应用程序想从自己选择​​的线程生成返回值时，可以返回`DeferredResult `。

* 当应用程序想要从线程池提交中产生值时，可以返回`ListenableFuture `或`CompletableFuture`/ `CompletionStage` 。

* 可以返回`ResponseBodyEmitter`以异步地将多个对象写入响应;也支持作为`ResponseEntity`内的主体。

* 可以返回`SseEmitter`以将异步的Server-Sent事件写入响应;也支持作为`ResponseEntity`内的主体。

* 可以返回`StreamingResponseBody`以异步写入响应`OutputStream`;也支持作为`ResponseEntity`内的主体

* 任何其他返回类型都被认为是要暴露给视图的单一模型属性，使用在方法级别（或基于返回类型类名称的默认属性名称）中通过`@ModelAttribute`指定的属性名称。该模型隐含地丰富了命令对象和`@ModelAttribute`注解引用数据访问器方法的结果。

#### 通过@RequestParam绑定请求参数到方法

使用`@RequestParam`注解将请求参数绑定到控制器中的方法参数。

以下代码片段显示用法：

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

默认情况下，使用此注解的参数是必需的，但您可以通过将`@RequestParam`的必需属性设置为`false`（例如`@RequestParam(name="id", required=false)`）来指定参数是可选的。

如果目标方法参数类型不是`String`，则会自动应用类型转换。 请参阅[the section called “Method Parameters And Type Conversion”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-ann-typeconversion)。

当在`Map<String，String>`或`MultiValueMap<String，String>`参数上使用`@RequestParam`注解时，映射将填充所有请求参数。

#### 使用@RequestBody注解映射请求体

`@RequestBody`方法参数注解表示方法参数应绑定到HTTP请求体的值。 例如：

```java
@PutMapping("/something")
public void handle(@RequestBody String body, Writer writer) throws IOException {
    writer.write(body);
}
```

通过使用`HttpMessageConverter`将请求体转换为method参数。 `HttpMessageConverter`负责将HTTP请求消息转换为对象，并从对象转换为HTTP响应体。 RequestMappingHandlerAdapter支持带有以下默认`HttpMessageConverters`的`@RequestBody`注解：

* `ByteArrayHttpMessageConverter`转换字节数组。

* `StringHttpMessageConverter`转换字符串。

* `FormHttpMessageConverter`将表单数据转换为MultiValueMap &lt;String，String&gt;。

* `SourceHttpMessageConverter`converts to/from a javax.xml.transform.Source.

有关这些转换器的更多信息，请参[Message Converters](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/remoting.html#rest-message-conversion)。 另请注意，如果使用MVC命名空间或MVC Java配置，默认情况下会注册更广泛的消息转换器。 有关详细信息，请参见[Section18.16.1, “Enabling the MVC Java Config or the MVC XML Namespace”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-config-enable)。 

如果您打算读写XML，则需要使用`org.springframework.oxm`包中的特定`Marshaller`和`Unmarshaller`实现配置`MarshallingHttpMessageConverter`。 下面的示例显示了如何直接在配置中执行此操作，但是如果您的应用程序通过MVC命名空间或MVC Java配置进行配置，请参见[Section18.16.1, “Enabling the MVC Java Config or the MVC XML Namespace”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-config-enable)。

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

`@RequestBody`方法参数可以用`@Valid`注解，在这种情况下，它将使用配置的`Validator`实例进行验证。 当使用MVC命名空间或MVC Java配置时，会自动配置一个JSR-303验证器，假设在类路径上可用JSR-303实现。

就像`@ModelAttribute`参数一样，可以使用一个`Errors`参数来检查错误。 如果未声明此类参数，则将引发`MethodArgumentNotValidException`异常。 该异常在`DefaultHandlerExceptionResolver`中处理，它将向客户端发送一个`400`错误。

| ![](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png) |
| :--- |
| 有关通过MVC命名空间或MVC Java配置配置消息转换器和验证器的信息，请参见[Section18.16.1, “Enabling the MVC Java Config or the MVC XML Namespace”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-config-enable)。 |

#### 使用@ResponseBody注解映射响应体

`@ResponseBody`注解类似于`@RequestBody`。 这个注解可以放在一个方法上，并且指示返回类型应该直接写入HTTP响应体（而不是放置在模型中，或者解释为视图名称）。 例如：

```java
@GetMapping("/something")
@ResponseBody
public String helloWorld() {
    return "Hello World";
}
```

上面的例子将会导致文本`Hello World`被写入到HTTP响应流中。

与`@RequestBody`一样，Spring通过使用`HttpMessageConverter`将返回的对象转换为响应正文。 有关这些转换器的更多信息，请参阅上一节和[Message Converters](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/remoting.html#rest-message-conversion)。

#### 使用@RestController注解创建REST控制器

控制器实现REST API是一种非常普通的用例，因此只能提供JSON，XML或自定义的MediaType内容。 为了方便起见，您可以使用`@RestController`注解您的控制器类，而不是使用`@ResponseBody`注解所有的`@RequestMapping`方法。

[`@RestController`](#)是一个结合了`@ResponseBody`和 `@Controller`的构造型注解。 更重要的是，它给你的控制器带来了更多的意义，也可能在框架的未来版本中带来额外的语义。

与常规`@Controller`s一样，`@RestController`可以由`@ControllerAdvice`或`@RestControllerAdvice` bean来协助。 有关详细信息，请参阅[the section called “Advising controllers with @ControllerAdvice and @RestControllerAdvice”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-ann-controller-advice)部分。

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

#### Using @ModelAttribute on a method

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

String-based values extracted from the request including request parameters, path variables, request headers, and cookie values may need to be converted to the target type of the method parameter or field \(e.g., binding a request parameter to a field in an`@ModelAttribute`parameter\) they’re bound to. If the target type is not`String`, Spring automatically converts to the appropriate type. All simple types such as int, long, Date, etc. are supported. You can further customize the conversion process through a`WebDataBinder`\(see[the section called “Customizing WebDataBinder initialization”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-ann-webdatabinder)\) or by registering`Formatters`with the`FormattingConversionService`\(see[Section 5.6, “Spring Field Formatting”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/validation.html#format)\).

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

