### 18.3.4异步请求处理

Spring MVC 3.2介绍了基于Servlet 3的异步请求处理。与往常一样，一个控制器方法现在可以返回一个`java.util.concurrent.Callable`并从Spring MVC管理的线程生成返回值，而不是返回一个 值。同时，主要的Servlet容器线程被退出并被释放并允许处理其他请求。Spring MVC`Callable`在一个单独的线程中调用一个单独的线程`TaskExecutor`，当`Callable`返回时，请求被分派回到Servlet容器，以使用返回的值来恢复处理`Callable`。这是一个这样一个控制器方法的例子：

```java
@PostMapping
public Callable<String> processUpload(final MultipartFile file) {

    return new Callable<String>() {
        public String call() throws Exception {
            // ...
            return "someView";
        }
    };

}
```

另一个选项是控制器方法返回一个实例`DeferredResult`。在这种情况下，返回值也将从任何线程生成，即不由Spring MVC管理的线程。例如，可以响应于诸如JMS消息，计划任务等的一些外部事件而产生结果。这是这样一个控制器方法的例子：

```java
@RequestMapping("/quotes")
@ResponseBody
public DeferredResult<String> quotes() {
    DeferredResult<String> deferredResult = new DeferredResult<String>();
    // Save the deferredResult somewhere..
    return deferredResult;
}

// In some other thread...
deferredResult.setResult(data);
```

没有任何Servlet 3.0异步请求处理功能的知识可能难以理解。这肯定有助于阅读。以下是有关基本机制的几个基本事实：

* 通过调用`request.startAsync()`，可以将`ServletRequest`置于异步模式。 这样做的主要效果是，Servlet以及任何过滤器都可以退出，但响应将保持打开状态，以便稍后完成处理。

* 可以用于进一步控制异步处理的`request.startAsync()`返回 调用`AsyncContext`。例如，它提供的方法`dispatch`类似于Servlet API中的转发，但它允许应用程序在Servlet容器线程上恢复请求处理。

* 在`ServletRequest`提供对当前`DispatcherType`可用于处理所述初始请求，一个异步调度，正向，以及其他的调度类型之间进行区分。

考虑到上述情况，以下是使用`Callable`进行异步请求处理的事件序列：

* 控制器返回一个`Callable`。

* Spring MVC开始异步处理，并将`Callable`提交给`TaskExecutor`，以便在单独的线程中进行处理。

* `DispatcherServlet`和所有Filter都退出Servlet容器线程，但响应保持打开状态。

* `Callable`生成一个结果，Spring MVC将请求发送回Servlet容器以恢复处理。

* `DispatcherServlet`被再次调用，处理从`Callable`异步产生的结果中恢复。



`DeferredResult`的序列非常相似，除非由应用程序生成任何线程的异步结果：

* 控制器返回一个`DeferredResult`并将其保存在某些内存中的队列或列表中，可以访问它。

* Spring MVC启动异步处理。

* `DispatcherServlet`和所有配置的Filter都退出请求处理线程，但响应保持打开状态。

* 应用程序从某个线程设置`DeferredResult`，Spring MVC将请求分派回Servlet容器。

* DispatcherServlet再次被调用，并且处理以异步产生的结果继续。

有关异步请求处理的动机的进一步背景，何时或为什么使用它，请阅读[此博客文章系列](https://spring.io/blog/2012/05/07/spring-mvc-3-2-preview-introducing-servlet-3-async-support)。

#### 异步请求异常处理

What happens if a`Callable`returned from a controller method raises an Exception while being executed? The short answer is the same as what happens when a controller method raises an exception. It goes through the regular exception handling mechanism. The longer explanation is that when a`Callable`raises an Exception Spring MVC dispatches to the Servlet container with the`Exception`as the result and that leads to resume request processing with the`Exception`instead of a controller method return value. When using a`DeferredResult`you have a choice whether to call`setResult`or`setErrorResult`with an`Exception`instance.

#### Intercepting Async Requests

A`HandlerInterceptor`can also implement`AsyncHandlerInterceptor`in order to implement the`afterConcurrentHandlingStarted`callback, which is called instead of`postHandle`and`afterCompletion`when asynchronous processing starts.

A`HandlerInterceptor`can also register a`CallableProcessingInterceptor`or a`DeferredResultProcessingInterceptor`in order to integrate more deeply with the lifecycle of an asynchronous request and for example handle a timeout event. See the Javadoc of`AsyncHandlerInterceptor`for more details.

The`DeferredResult`type also provides methods such as`onTimeout(Runnable)`and`onCompletion(Runnable)`. See the Javadoc of`DeferredResult`for more details.

When using a`Callable`you can wrap it with an instance of`WebAsyncTask`which also provides registration methods for timeout and completion.

#### HTTP Streaming

A controller method can use`DeferredResult`and`Callable`to produce its return value asynchronously and that can be used to implement techniques such as[long polling](https://spring.io/blog/2012/05/08/spring-mvc-3-2-preview-techniques-for-real-time-updates/)where the server can push an event to the client as soon as possible.

What if you wanted to push multiple events on a single HTTP response? This is a technique related to "Long Polling" that is known as "HTTP Streaming". Spring MVC makes this possible through the`ResponseBodyEmitter`return value type which can be used to send multiple Objects, instead of one as is normally the case with`@ResponseBody`, where each Object sent is written to the response with an`HttpMessageConverter`.

Here is an example of that:

```java
RequestMapping("/events")
public ResponseBodyEmitter handle() {
    ResponseBodyEmitter emitter = new ResponseBodyEmitter();
    // Save the emitter somewhere..
    return emitter;
}

// In some other thread
emitter.send("Hello once");

// and again later on
emitter.send("Hello again");

// and done at some point
emitter.complete();
```

Note that`ResponseBodyEmitter`can also be used as the body in a`ResponseEntity`in order to customize the status and headers of the response.

#### HTTP Streaming With Server-Sent Events

`SseEmitter`is a sub-class of`ResponseBodyEmitter`providing support for[Server-Sent Events](https://www.w3.org/TR/eventsource/). Server-sent events is a just another variation on the same "HTTP Streaming" technique except events pushed from the server are formatted according to the W3C Server-Sent Events specification.

Server-Sent Events can be used for their intended purpose, that is to push events from the server to clients. It is quite easy to do in Spring MVC and requires simply returning a value of type`SseEmitter`.

Note however that Internet Explorer does not support Server-Sent Events and that for more advanced web application messaging scenarios such as online games, collaboration, financial applicatinos, and others it’s better to consider Spring’s WebSocket support that includes SockJS-style WebSocket emulation falling back to a very wide range of browsers \(including Internet Explorer\) and also higher-level messaging patterns for interacting with clients through a publish-subscribe model within a more messaging-centric architecture. For further background on this see[the following blog post](http://blog.pivotal.io/pivotal/products/websocket-architecture-in-spring-4-0).

#### HTTP Streaming Directly To The OutputStream

`ResponseBodyEmitter`allows sending events by writing Objects to the response through an`HttpMessageConverter`. This is probably the most common case, for example when writing JSON data. However sometimes it is useful to bypass message conversion and write directly to the response`OutputStream`for example for a file download. This can be done with the help of the`StreamingResponseBody`return value type.

Here is an example of that:

```java
@RequestMapping("/download")
public StreamingResponseBody handle() {
    return new StreamingResponseBody() {
        @Override
        public void writeTo(OutputStream outputStream) throws IOException {
            // write...
        }
    };
}
```

Note that`StreamingResponseBody`can also be used as the body in a`ResponseEntity`in order to customize the status and headers of the response.

#### Configuring Asynchronous Request Processing

##### Servlet Container Configuration

For applications configured with a`web.xml`be sure to update to version 3.0:

```java
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            http://java.sun.com/xml/ns/javaee
            http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
    version="3.0">

    ...

</web-app>
```

Asynchronous support must be enabled on the`DispatcherServlet`through the`true`sub-element in`web.xml`. Additionally any`Filter`that participates in asyncrequest processing must be configured to support the ASYNC dispatcher type. It should be safe to enable the ASYNC dispatcher type for all filters provided with the Spring Framework since they usually extend`OncePerRequestFilter`and that has runtime checks for whether the filter needs to be involved in async dispatches or not.

Below is some example web.xml configuration:

```java
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
            http://java.sun.com/xml/ns/javaee
            http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
    version="3.0">

    <filter>
        <filter-name>Spring OpenEntityManagerInViewFilter</filter-name>
        <filter-class>org.springframework.~.OpenEntityManagerInViewFilter</filter-class>
        <async-supported>true</async-supported>
    </filter>

    <filter-mapping>
        <filter-name>Spring OpenEntityManagerInViewFilter</filter-name>
        <url-pattern>/*</url-pattern>
        <dispatcher>REQUEST</dispatcher>
        <dispatcher>ASYNC</dispatcher>
    </filter-mapping>

</web-app>
```

If using Servlet 3, Java based configuration for example via`WebApplicationInitializer`, you’ll also need to set the "asyncSupported" flag as well as the ASYNC dispatcher type just like with`web.xml`. To simplify all this configuration, consider extending`AbstractDispatcherServletInitializer`, or better`AbstractAnnotationConfigDispatcherServletInitializer`which automatically set those options and make it very easy to register`Filter`instances.

##### Spring MVC Configuration

The MVC Java config and the MVC namespace provide options for configuring asynchronous request processing.`WebMvcConfigurer`has the method`configureAsyncSupport`while`<mvc:annotation-driven>`has an`<async-support>`sub-element.

Those allow you to configure the default timeout value to use for async requests, which if not set depends on the underlying Servlet container \(e.g. 10 seconds on Tomcat\). You can also configure an`AsyncTaskExecutor`to use for executing`Callable`instances returned from controller methods. It is highly recommended to configure this property since by default Spring MVC uses`SimpleAsyncTaskExecutor`. The MVC Java config and the MVC namespace also allow you to register`CallableProcessingInterceptor`and`DeferredResultProcessingInterceptor`instances.

If you need to override the default timeout value for a specific`DeferredResult`, you can do so by using the appropriate class constructor. Similarly, for a`Callable`, you can wrap it in a`WebAsyncTask`and use the appropriate class constructor to customize the timeout value. The class constructor of`WebAsyncTask`also allows providing an`AsyncTaskExecutor`.

