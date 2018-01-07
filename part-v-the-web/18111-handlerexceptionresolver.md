### 18.11.1 HandlerExceptionResolver

Spring `HandlerExceptionResolver`实现处理控制器执行期间发生的意外异常。 `HandlerExceptionResolver`有点类似于你可以在web应用程序描述符`web.xml`中定义的异常映射。 但是，他们提供了一个更灵活的方式来做到这一点。 例如，它们提供有关抛出异常时正在执行哪个处理程序的信息。 此外，处理异常的编程方式为您提供了更多的选择，可以在将请求转发到另一个URL（与使用特定于Servlet的异常映射相同的最终结果）之前做出适当的响应。

除了实现`HandlerExceptionResolver`接口（这只是实现`resolveException(Exception, Handler)`方法并返回一个`ModelAndView`）之外，还可以使用提供的`SimpleMappingExceptionResolver`或创建`@ExceptionHandler`方法。 `SimpleMappingExceptionResolver`使您可以将可能引发的任何异常的类名称映射到视图名称。 这在功能上等同于Servlet API的异常映射功能，但是也可以从不同的处理程序实现更多细化的异常映射。 另一方面，`@ExceptionHandler`注解可用于应该被调用来处理异常的方法。 这样的方法可以在`@Controller`中本地定义，也可以在`@ControllerAdvice`类中定义时应用于许多`@Controller`类。 以下部分更详细地解释了这一点。

