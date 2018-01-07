### 18.11.3处理标准的Spring MVC异常

在处理请求时，Spring MVC可能会引发一些异常。 `SimpleMappingExceptionResolver`可以根据需要轻松地将任何异常映射到默认错误视图。 但是，在使用以自动方式解释响应的客户端时，您需要在响应中设置特定的状态码。 根据引发的异常，状态码可能表示客户端错误（4xx）或服务器错误（5xx）。

`DefaultHandlerExceptionResolver`将Spring MVC异常转换为特定的错误状态代码。 默认情况下，MVC命名空间，MVC Java配置以及`DispatcherServlet`（即不使用MVC命名空间或Java配置时）都被注册。 下面列出的是这个解析器处理的一些异常和相应的状态码：

| Exception | HTTP Status Code |
| :--- | :--- |
| `BindException` | 400 \(Bad Request\) |
| `ConversionNotSupportedException` | 500 \(Internal Server Error\) |
| `HttpMediaTypeNotAcceptableException` | 406 \(Not Acceptable\) |
| `HttpMediaTypeNotSupportedException` | 415 \(Unsupported Media Type\) |
| `HttpMessageNotReadableException` | 400 \(Bad Request\) |
| `HttpMessageNotWritableException` | 500 \(Internal Server Error\) |
| `HttpRequestMethodNotSupportedException` | 405 \(Method Not Allowed\) |
| `MethodArgumentNotValidException` | 400 \(Bad Request\) |
| `MissingPathVariableException` | 500 \(Internal Server Error\) |
| `MissingServletRequestParameterException` | 400 \(Bad Request\) |
| `MissingServletRequestPartException` | 400 \(Bad Request\) |
| `NoHandlerFoundException` | 404 \(Not Found\) |
| `NoSuchRequestHandlingMethodException` | 404 \(Not Found\) |
| `TypeMismatchException` | 400 \(Bad Request\) |

`DefaultHandlerExceptionResolver`通过设置响应的状态透明地工作。 但是，当应用程序可能需要为每个错误响应添加对开发人员友好的内容（例如，提供REST API时）时，它不会将任何错误内容写入响应主体。 您可以通过视图解析来准备模型和视图并呈现错误内容，即通过配置`ContentNegotiatingViewResolver`，`MappingJackson2JsonView`等。 但是，您可能更喜欢使用`@ExceptionHandler`方法。

如果您更喜欢通过`@ExceptionHandler`方法编写错误内容，则可以扩展`ResponseEntityExceptionHandler`。 这是@ControllerAdvice类的便利基础，它提供了一个`@ExceptionHandler`方法来处理标准的Spring MVC异常并返回`ResponseEntity`。 这允许您自定义响应并使用消息转换器编写错误内容。 有关更多详细信息，请参阅`ResponseEntityExceptionHandler` javadocs。



