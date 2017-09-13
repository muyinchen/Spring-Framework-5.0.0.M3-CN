### 18.11.4 REST Controller Exception Handling

An`@RestController`may use`@ExceptionHandler`methods that return a`ResponseEntity`to provide both a response status and error details in the body of the response. Such methods may also be added to`@ControllerAdvice`classes for exception handling across a subset or all controllers.

A common requirement is to include error details in the body of the response. Spring does not automatically do this \(although Spring Boot does\) because the representation of error details in the response body is application specific.

Applications that wish to implement a global exception handling strategy with error details in the response body should consider extending the abstract base class`ResponseEntityExceptionHandler`which provides handling for the exceptions that Spring MVC raises and provides hooks to customize the response body as well as to handle other exceptions. Simply declare the extension class as a Spring bean and annotate it with`@ControllerAdvice`. For more details see See`ResponseEntityExceptionHandler`.

