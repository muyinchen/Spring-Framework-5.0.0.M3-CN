### 18.11.5 Annotating Business Exceptions With @ResponseStatus

A business exception can be annotated with`@ResponseStatus`. When the exception is raised, the`ResponseStatusExceptionResolver`handles it by setting the status of the response accordingly. By default the`DispatcherServlet`registers the`ResponseStatusExceptionResolver`and it is available for use.

