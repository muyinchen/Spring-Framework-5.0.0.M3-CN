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

* 如果该方法用`@ResponseBody`注解，则返回类型将写入响应HTTP主体。返回值将使用`HttpMessageConverters`转换为声明的方法参数类型。请参阅 [the section called “Mapping the response body with the @ResponseBody annotation”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-ann-responsebody)。

* 一个`HttpEntity`或`ResponseEntity`对象来提供对Servlet响应HTTP头和内容的访问。实体将使用`HttpMessageConverters`转换为响应流。请参阅[the section called “Using HttpEntity”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-ann-httpentity).

* 一个`HttpHeaders`对象返回没有正文的响应。

* 当应用程序想要在由Spring MVC管理的线程中异步生成返回值时，可以返回`Callable`。

* 当应用程序想从自己选择​​的线程生成返回值时，可以返回`DeferredResult`。

* 当应用程序想要从线程池提交中产生值时，可以返回`ListenableFuture`或`CompletableFuture`/ `CompletionStage` 。

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

#### 使用HttpEntity

这`HttpEntity`是相似的`@RequestBody`和`@ResponseBody`。除了访问请求和响应主体之外`HttpEntity`（和响应特定子类`ResponseEntity`）还允许访问请求和响应头，如下所示：

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

上述示例获取`MyRequestHeader`请求标头的值，并将其作为字节数组读取。它将`MyResponseHeader`响应添加到`Hello World`响应流中，并将响应状态代码设置为201（已创建）。

至于`@RequestBody`和`@ResponseBody`，Spring使用`HttpMessageConverter`从和请求和响应流转换。有关这些转换器的更多信息，请参阅上一部分和[消息转换器](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/remoting.html#rest-message-conversion)。

#### 在方法上使用@ModelAttribute

该`@ModelAttribute`注解可以对方法或方法的参数来使用。本节将介绍其在方法上的用法，下一节将介绍其在方法参数上的用法。

方法上的`@ModelAttribute`指示该方法的目的是添加一个或多个模型属性。 这些方法支持与`@RequestMapping`方法相同的参数类型，但不能直接映射到请求。 相反，控制器中的`@ModelAttribute`方法在相同控制器内的`@RequestMapping`方法之前被调用。 几个例子：

```java
//添加一个属性
//该方法的返回值被添加到名为“account”的模型中
//您可以通过@ModelAttribute（“myAccount”）

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

`@ModelAttribute`方法用于填充具有常用属性的模型，例如使用状态或宠物类型填充下拉列表，或者检索诸如Account的命令对象，以便使用它来表示HTML表单上的数据。后一种情况在下一节进一步讨论。

注意两种风格的`@ModelAttribute`方法。在第一个方法中，该方法通过返回它隐式地添加一个属性。在第二个方法中，该方法接受`Model`并添加任意数量的模型属性。您可以根据需要选择两种风格。

控制器可以有多种`@ModelAttribute`方法。所有这些方法都`@RequestMapping`在相同控制器的方法之前被调用。

`@ModelAttribute`方法也可以在一个`@ControllerAdvice`注释类中定义，并且这种方法适用于许多控制器。有关更多详细信息，请参阅[“使用@ControllerAdvice和@RestControllerAdvice建议控制器”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-ann-controller-advice)一节。

| ![](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/tip.png) |
| :--- |
| 当没有明确指定模型属性名称时会发生什么？在这种情况下，根据其类型将默认名称分配给模型属性。例如，如果该方法返回类型的对象`Account`，则使用的默认名称为“account”。您可以通过`@ModelAttribute`注释的值更改它。如果直接添加属性`Model`，请使用适当的重载`addAttribute(..)`方法 – 即，带有或不带有属性名称。 |

该`@ModelAttribute`注解可在使用`@RequestMapping`方法为好。在这种情况下，`@RequestMapping`方法的返回值将被解释为模型属性而不是视图名称。视图名称是基于视图名称约定导出的，非常类似于返回的方法`void`- 请参见[第18.13.3节“View – RequestToViewNameTranslator”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-coc-r2vnt)。

#### 在方法参数上使用@ModelAttribute

如上一节所述`@ModelAttribute`，可以在方法或方法参数上使用。本节介绍了其在方法参数中的用法。

一个`@ModelAttribute`上的方法参数指示参数应该从模型中检索。如果模型中不存在，参数首先被实例化，然后添加到模型中。一旦出现在模型中，参数的字段应该从具有匹配名称的所有请求参数中填充。这被称为Spring MVC中的数据绑定，这是一种非常有用的机制，可以节省您逐个解析每个表单字段。

```java
@PostMapping("/owners/{ownerId}/pets/{petId}/edit")
public String processSubmit(@ModelAttribute Pet pet) { }
```

鉴于上述例子，Pet实例可以从哪里来？有几个选择：

* 由于使用`@SessionAttributes`- 可能已经在模型中- 请参阅[“使用@SessionAttributes将模型属性存储在请求之间的HTTP会话中”一节](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-ann-sessionattrib)。

* 由于`@ModelAttribute`在同一控制器中的方法，它可能已经在模型中 – 如上一节所述。

* 它可以基于URI模板变量和类型转换器（下面更详细地解释）来检索。

* 它可以使用其默认构造函数实例化。

一种`@ModelAttribute`方法是从数据库中检索属性的常用方法，可以通过使用可选地在请求之间存储属性`@SessionAttributes`。在某些情况下，通过使用URI模板变量和类型转换器来检索属性可能很方便。这是一个例子：

```java
@PutMapping("/accounts/{account}")
public String save(@ModelAttribute("account") Account account) {
    // ...
}
```

在此示例中，模型属性（即“account”）的名称与URI模板变量的名称相匹配。如果您注册`Converter`，可以将`String`帐户值转换为一个`Account`实例，则上述示例将无需使用`@ModelAttribute`方法。

下一步是数据绑定。该`WebDataBinder`级比赛要求参数名称-包括查询字符串参数和表单域-以模拟通过名称属性字段。在必要时已经应用了类型转换（从字符串到目标字段类型）之后填充匹配字段。数据绑定和验证在[第5章验证，数据绑定和类型转换中介绍](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/validation.html)。自定义控制器级别的数据绑定过程将在[“自定义WebDataBinder初始化”一节中介绍](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-ann-webdatabinder)。

由于数据绑定，可能会出现错误，例如缺少必填字段或类型转换错误。要检查这些错误，请在`BindingResult`参数后立即添加一个`@ModelAttribute`参数：

```java
@PostMapping("/owners/{ownerId}/pets/{petId}/edit")
public String processSubmit(@ModelAttribute("pet") Pet pet, BindingResult result) {

    if (result.hasErrors()) {
        return "petForm";
    }

    // ...

}
```

使用一个`BindingResult`你可以检查是否发现错误，在这种情况下，渲染相同的形式通常是在Spring的`<errors>`表单标签的帮助下显示错误的。

请注意，在某些情况下，在没有数据绑定的情况下获取模型中的属性可能是有用的。对于这种情况，您可以将其注入`Model`控制器，或者使用注释上的`binding`标志：

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

除了数据绑定之外，您还可以使用自己的自定义验证器调用验证，传递与`BindingResult`用于记录数据绑定错误相同的验证器。这允许在一个地方累积数据绑定和验证错误，并随后向用户报告：

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

或者您可以通过添加JSR-303`@Valid`注解自动调用验证：

```java
@PostMapping("/owners/{ownerId}/pets/{petId}/edit")
public String processSubmit(@Valid @ModelAttribute("pet") Pet pet, BindingResult result) {

    if (result.hasErrors()) {
        return "petForm";
    }

    // ...

}
```

有关如何配置和使用验证的详细信息，请参见[第5.8节“Spring验证”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/validation.html#validation-beanvalidation)和[第5章验证，数据绑定和类型](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/validation.html)转换。

#### 使用@SessionAttributes将模型属性存储在请求之间的HTTP会话中

类型级`@SessionAttributes`注释声明特定处理程序使用的会话属性。这通常将列出模型属性或模型属性的类型，这些模型属性或类型应该透明地存储在会话或某些会话存储中，作为后续请求之间的格式支持bean。

以下代码片段显示了此注解的用法，指定了模型属性名称：

```java
@Controller
@RequestMapping("/editPet.do")
@SessionAttributes("pet")
public class EditPetForm {
    // ...
}
```

#### 使用@SessionAttribute访问预先存在的全局会话属性

如果您需要访问全局管理的预先存在的会话属性，即控制器外部（例如，通过过滤器），并且可能存在或可能不存在，`@SessionAttribute`则会使用方法参数上的注解：

```java
@RequestMapping("/")
public String handle(@SessionAttribute User user) {
    // ...
}
```

对于需要添加或删除会话属性的用例，请考虑注入`org.springframework.web.context.request.WebRequest`或`javax.servlet.http.HttpSession`控制方法。

为了在会话中临时存储模型属性作为控制器工作流的一部分，请考虑使用[“使用@SessionAttributes将模型属性存储在请求之间的HTTP会话中”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-ann-sessionattrib)`SessionAttributes`中[所述的一节](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-ann-sessionattrib)。

#### 使用@RequestAttribute来访问请求属性

到类似`@SessionAttribute`的`@RequestAttribute`注解可以被用于访问由滤波器或拦截器创建的预先存在的请求属性：

```java
@RequestMapping("/")
public String handle(@RequestAttribute Client client) {
    // ...
}
```

#### 使用“application / x-www-form-urlencoded”数据

以前的章节介绍了`@ModelAttribute`如何支持浏览器客户端的表单提交请求。建议与非浏览器客户端的请求一起使用相同的注解。然而，在使用HTTP PUT请求时，有一个显着的区别。浏览器可以通过HTTP GET或HTTP POST提交表单数据。非浏览器客户端也可以通过HTTP PUT提交表单。这提出了一个挑战，因为Servlet规范要求`ServletRequest.getParameter*()`一系列方法仅支持HTTP POST的表单域访问，而不支持HTTP PUT。

为了支持HTTP PUT和PATCH请求，该`spring-web`模块提供了`HttpPutFormContentFilter`可以在以下配置中的过滤器`web.xml`：

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

上述过滤器拦截具有内容类型的HTTP PUT和PATCH请求`application/x-www-form-urlencoded`，从请求的正文 中读取表单数据，并包装`ServletRequest`以便通过`ServletRequest.getParameter*()`一系列方法使表单数据可用 。

| ![](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png) |
| :--- |
| 由于`HttpPutFormContentFilter`消耗了请求的正文，因此不应配置为依赖其他转换器的PUT或PATCH URL `application/x-www-form-urlencoded`。这包括`@RequestBody MultiValueMap`and`HttpEntity>`。 |

#### 使用@CookieValue注解映射Cookie值

该`@CookieValue`注解允许将方法参数绑定到HTTP cookie的值。

让我们考虑以下cookie已被接收到http请求：

```java
JSESSIONID=415A4AC178C59DACE0B2C9CA727CDD84
```

以下代码示例演示如何获取`JSESSIONID`cookie 的值：

```java
@RequestMapping("/displayHeaderInfo.do")
public void displayHeaderInfo(@CookieValue("JSESSIONID") String cookie) {
    //...
}
```

如果目标方法参数类型不是，则会自动应用类型转换`String`。请参阅[“方法参数和类型转换”一节](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-ann-typeconversion)。

#### 使用@RequestHeader注解映射请求标头属性

该`@RequestHeader`注解允许将一个方法参数绑定到请求头。

以下是一个示例请求标头：

```java
Host                    localhost:8080
Accept                  text/html,application/xhtml+xml,application/xml;q=0.9
Accept-Language         fr,en-gb;q=0.7,en;q=0.3
Accept-Encoding         gzip,deflate
Accept-Charset          ISO-8859-1,utf-8;q=0.7,*;q=0.7
Keep-Alive              300
```

以下代码示例演示了如何获取`Accept-Encoding`和`Keep-Alive`标题的值：

```java
@RequestMapping("/displayHeaderInfo.do")
public void displayHeaderInfo(@RequestHeader("Accept-Encoding") String encoding,
        @RequestHeader("Keep-Alive") long keepAlive) {
    //...
}
```

如果方法参数不是`String`，则会自动应用类型转换。 查看[the section called “Method Parameters And Type Conversion”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-ann-typeconversion)一节。

在`Map`，`MultiValueMap`或`HttpHeaders`参数上使用`@RequestHeader`注解时，映射将填充所有标题值。

| ![](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/tip.png) |
| :--- |
| 内置支持可用于将逗号分隔的字符串转换为字符串或类型转换系统已知的其他类型的数组/集合。例如，注解的方法参数`@RequestHeader("Accept")`可以是类型`String`，也可以是`String[]`或`List`。 |

#### 方法参数和类型转换

从请求中提取的基于字符串的值（包括请求参数，路径变量，请求头和cookie值）可能需要转换为方法参数或字段的目标类型（例如，将请求参数绑定到参数中的字段`@ModelAttribute`）他们一定会。如果目标类型不是`String`，Spring将自动转换为相应的类型。支持所有简单的类型，如int，long，Date等。您可以进一步自定义通过转换过程`WebDataBinder`（见[称为“定制WebDataBinder初始化”一节](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-ann-webdatabinder)），或者通过注册`Formatters`与`FormattingConversionService`（参见[5.6节，“春字段格式”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/validation.html#format)）。

#### 自定义WebDataBinder初始化

要通过Spring定制与PropertyEditor的请求参数绑定`WebDataBinder`，可以使用`@InitBinder`控制器中的-annotated`@InitBinder`方法，`@ControllerAdvice`类中的方法或提供自定义`WebBindingInitializer`。有关更多详细信息，请参阅[“使用@ControllerAdvice和@RestControllerAdvice建议控制器”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-ann-controller-advice)一节。

##### 使用@InitBinder自定义数据绑定

Annotating controller methods with`@InitBinder`allows you to configure web data binding directly within your controller class.`@InitBinder`identifies methods that initialize the`WebDataBinder`that will be used to populate command and form object arguments of annotated handler methods.

Such init-binder methods support all arguments that`@RequestMapping`methods support, except for command/form objects and corresponding validation result objects. Init-binder methods must not have a return value. Thus, they are usually declared as`void`. Typical arguments include`WebDataBinder`in combination with`WebRequest`or`java.util.Locale`, allowing code to register context-specific editors.

The following example demonstrates the use of`@InitBinder`to configure a`CustomDateEditor`for all`java.util.Date`form properties.

注解控制器方法，`@InitBinder`允许您直接在控制器类中配置Web数据绑定。`@InitBinder`识别用于初始化`WebDataBinder`将用于填充命名和表示注解处理程序方法的对象参数的方法。

这种init-binder方法支持方法支持的所有参数`@RequestMapping`，除了命令/表单对象和相应的验证结果对象。Init-binder方法不能有返回值。因此，它们通常被声明为`void`。典型的参数包括`WebDataBinder`与`WebRequest`或者`java.util.Locale`允许代码注册上下文相关的编辑器。

以下示例演示如何使用`@InitBinder`为所有`java.util.Date`表单属性配置`CustomDateEditor`。

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

或者，从Spring 4.2起，考虑使用`addCustomFormatter`来指定`Formatter`实现而不是`PropertyEditor`实例。如果您碰巧`Formatter`在共享`FormattingConversionService`中安装一个基于安装程序的 方法，那么特别有用的方法可以重用于控制器特定的绑定规则调整。

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

##### 配置自定义WebBindingInitializer

要外部化数据绑定初始化，您可以提供`WebBindingInitializer`接口的自定义实现，然后通过为其提供自定义bean配置来启用`RequestMappingHandlerAdapter`，从而覆盖默认配置。

PetClinic应用程序中的以下示例显示了使用该接口的自定义实现的`WebBindingInitializer`配置`org.springframework.samples.petclinic.web.ClinicBindingInitializer`，它配置了几个PetClinic控制器所需的PropertyEditor。

```java
<bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter">
    <property name="cacheSeconds" value="0"/>
    <property name="webBindingInitializer">
        <bean class="org.springframework.samples.petclinic.web.ClinicBindingInitializer"/>
    </property>
</bean>
```

`@InitBinder`方法也可以在`@ControllerAdvice`注解类中定义，在这种情况下，它们适用于匹配的控制器。 这提供了使用`WebBindingInitializer`的替代方法。有关更多详细信息，请参阅[“使用@ControllerAdvice和@RestControllerAdvice建议控制器”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-ann-controller-advice)一节。

#### 通过@ControllerAdvice和@RestControllerAdvice为控制器提供建议

`@ControllerAdvice`注解是一个组件注解允许实现类自动检测通过类路径扫描。当使用MVC命名空间或MVC Java配置时，它将自动启用。

使用`@ControllerAdvice`注解的类可以包含`@ExceptionHandler`，`@InitBinder`和`@ModelAttribute`注解的方法，这些方法将适用于`@RequestMapping`所有控制器的层次结构的方法，而不是内声明它们控制器层次。

`@RestControllerAdvice`是`@ExceptionHandler`方法默认使用`@ResponseBody`语义的替代方案。

`@ControllerAdvice`和`@RestControllerAdvice`都可以定位控制器的一个子集：

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

查看[`@ControllerAdvice`文档](http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/web/bind/annotation/ControllerAdvice.html)了解更多详细信息。

#### Jackson序列化视图支持

有时将内容过滤将被序列化到HTTP响应主体的对象有时是有用的。为了提供这样的功能，Spring MVC内置了对[Jackson的Serialization Views](http://wiki.fasterxml.com/JacksonJsonViews)进行渲染的支持。

要将它用于返回`ResponseEntity`的`@ResponseBody`控制器方法或控制器方法，只需添加`@JsonView`注解，并指定要使用的视图类或接口的类参数即可：

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

