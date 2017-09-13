### 18.11.3Handling Standard Spring MVC Exceptions

Spring MVC may raise a number of exceptions while processing a request. The`SimpleMappingExceptionResolver`can easily map any exception to a default error view as needed. However, when working with clients that interpret responses in an automated way you will want to set specific status code on the response. Depending on the exception raised the status code may indicate a client error \(4xx\) or a server error \(5xx\).

The`DefaultHandlerExceptionResolver`translates Spring MVC exceptions to specific error status codes. It is registered by default with the MVC namespace, the MVC Java config, and also by the`DispatcherServlet`\(i.e. when not using the MVC namespace or Java config\). Listed below are some of the exceptions handled by this resolver and the corresponding status codes:

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

The`DefaultHandlerExceptionResolver`works transparently by setting the status of the response. However, it stops short of writing any error content to the body of the response while your application may need to add developer-friendly content to every error response for example when providing a REST API. You can prepare a`ModelAndView`and render error content through view resolution — i.e. by configuring a`ContentNegotiatingViewResolver`,`MappingJackson2JsonView`, and so on. However, you may prefer to use`@ExceptionHandler`methods instead.

If you prefer to write error content via`@ExceptionHandler`methods you can extend`ResponseEntityExceptionHandler`instead. This is a convenient base for`@ControllerAdvice`classes providing an`@ExceptionHandler`method to handle standard Spring MVC exceptions and return`ResponseEntity`. That allows you to customize the response and write error content with message converters. See the`ResponseEntityExceptionHandler`javadocs for more details.

