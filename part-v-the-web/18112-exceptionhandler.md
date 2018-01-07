### 18.11.2@ExceptionHandler

`HandlerExceptionResolver`接口和`SimpleMappingExceptionResolver`实现允许您在转发到这些视图之前，将Exceptions和一些可选的Java逻辑声明性地映射到特定的视图。 但是，在某些情况下，特别是依赖`@ResponseBody`方法而不是视图分辨率时，直接设置响应的状态并可选地将错误内容写入响应的主体可能更方便。

你可以用`@ExceptionHandler`方法来做到这一点。 在控制器中声明时，这些方法适用于该控制器（或其任何子类）的`@RequestMapping`方法引发的异常。 您也可以在`@ControllerAdvice`类中声明一个`@ExceptionHandler`方法，在这种情况下，它可以处理来自许多控制器的`@RequestMapping`方法的异常。 下面是一个controller-local `@ExceptionHandler`方法的例子：

```java
@Controller
public class SimpleController {

    // @RequestMapping methods omitted ...

    @ExceptionHandler(IOException.class)
    public ResponseEntity<String> handleIOException(IOException ex) {
        // prepare responseEntity
        return responseEntity;
    }

}
```

`@ExceptionHandler`值可以设置为一个Exception类型的数组。 如果抛出一个与列表中的某个类型匹配的异常，那么将调用用匹配的`@ExceptionHandler`注解的方法。 如果未设置注释值，则使用列为方法参数的异常类型。

就像使用`@RequestMapping`注解标注的标准控制器方法一样，`@ExceptionHandler`方法的方法参数和返回值可以是灵活的。 例如，可以在Servlet环境中访问`HttpServletRequest`。 返回类型可以是`String`，它被解释为一个视图名称，一个`ModelAndView`对象，一个`ResponseEntity`，或者你也可以添加`@ResponseBody`方法返回值与消息转换器转换并写入响应流。

