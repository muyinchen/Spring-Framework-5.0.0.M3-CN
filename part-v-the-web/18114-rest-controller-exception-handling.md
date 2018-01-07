### 18.11.4 REST控制器异常处理

`@RestController`可以使用`@ExceptionHandler`方法返回一个`ResponseEntity`，在响应的主体中提供响应状态和错误细节。 这些方法也可以添加到`@ControllerAdvice`类中，以处理子集或所有控制器的异常处理。

一个常见的要求是在响应的主体中包含错误细节。 Spring不会自动执行此操作（虽然Spring Boot会这样做），因为响应主体中错误详细信息的表示形式是特定于应用程序的。

希望在响应主体中实现具有错误细节的全局异常处理策略的应用程序应考虑扩展抽象基类`ResponseEntityExceptionHandler`，该类提供对Spring MVC引发的异常的处理，并提供挂钩来自定义响应主体以及处理其他异常。 只需将扩展类声明为Spring bean，并使用`@ControllerAdvice`对其进行注释。 有关更多详细信息，请参阅请参阅`ResponseEntityExceptionHandler`。



