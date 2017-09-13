### 18.11.2@ExceptionHandler

The`HandlerExceptionResolver`interface and the`SimpleMappingExceptionResolver`implementations allow you to map Exceptions to specific views declaratively along with some optional Java logic before forwarding to those views. However, in some cases, especially when relying on`@ResponseBody`methods rather than on view resolution, it may be more convenient to directly set the status of the response and optionally write error content to the body of the response.

You can do that with`@ExceptionHandler`methods. When declared within a controller such methods apply to exceptions raised by`@RequestMapping`methods of that controller \(or any of its sub-classes\). You can also declare an`@ExceptionHandler`method within an`@ControllerAdvice`class in which case it handles exceptions from`@RequestMapping`methods from many controllers. Below is an example of a controller-local`@ExceptionHandler`method:

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

The`@ExceptionHandler`value can be set to an array of Exception types. If an exception is thrown that matches one of the types in the list, then the method annotated with the matching`@ExceptionHandler`will be invoked. If the annotation value is not set then the exception types listed as method arguments are used.

Much like standard controller methods annotated with a`@RequestMapping`annotation, the method arguments and return values of`@ExceptionHandler`methods can be flexible. For example, the`HttpServletRequest`can be accessed in Servlet environments. The return type can be a`String`, which is interpreted as a view name, a`ModelAndView`object, a`ResponseEntity`, or you can also add the`@ResponseBody`to have the method return value converted with message converters and written to the response stream.

