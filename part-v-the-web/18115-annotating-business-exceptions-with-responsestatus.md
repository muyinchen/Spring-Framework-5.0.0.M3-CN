### 18.11.5 使用@ResponseStatus注解业务异常

业务异常可以使用`@ResponseStatus`进行注释。 当引发异常时，`ResponseStatusExceptionResolver`通过相应地设置响应的状态来处理它。 默认情况下，`DispatcherServlet`注册`ResponseStatusExceptionResolver`，并可供使用。

