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

如果从控制器方法返回的`Callable`在执行时引发异常，会发生什么情况？ 简短的答案与控制器方法引发异常时发生的情况相同。 它经历了常规的异常处理机制。 更长的解释是，当`Callable`引发异常Spring MVC调度到Servlet容器与异常的结果，并导致恢复请求处理与异常，而不是一个控制器方法的返回值。 当使用`DeferredResources`时，您可以选择是否使用`Exception`实例调用`setResult`或`setErrorResult`。

#### 拦截异步请求

`HandlerInterceptor`还可以实现`AsyncHandlerInterceptor`以实现`afterConcurrentHandlingStarted`回调，在异步处理启动时调用，而不是`postHandle`和`afterCompletion`。

HandlerInterceptor也可以注册一个`CallableProcessingInterceptor`或者一个`DeferredResultProcessingInterceptor`，以便更深入地集成异步请求的生命周期，例如处理超时事件。 有关更多详细信息，请参阅`AsyncHandlerInterceptor`的Javadoc。

`DeferredResult`类型还提供诸如`onTimeout(Runnable)`和`onCompletion(Runnable)`等方法。 有关更多详细信息，请参阅`DeferredResult`的Javadoc。

使用`Callable`时，可以用`WebAsyncTask`的实例包装它，该实例还提供了超时和完成的注册方法。

#### HTTP 流式传输

控制器方法可以异步地使用`DeferredResult`和`Callable`产生其返回值，并且可以用于实现诸如[长轮询的技术](https://spring.io/blog/2012/05/08/spring-mvc-3-2-preview-techniques-for-real-time-updates/)，其中服务器可以尽快将事件推送到客户端。

如果您想在单个HTTP响应中推送多个事件怎么办？这是一个与“长查询”相关的技术，被称为“HTTP流”。Spring MVC可以通过`ResponseBodyEmitter`可以用于发送多个对象的返回值类型来实现，而不是像通常情况那样`@ResponseBody`发送的对象，其中每个发送的对象都被写入到响应中`HttpMessageConverter`。

这是一个例子：

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

请注意，`ResponseBodyEmitter`也可以用作`ResponseEntity`中的主体，以便自定义响应的状态和标题。

#### HTTP流与服务器发送的事件

SseEmitter是ResponseBodyEmitter的一个子类，为[服务器发送事件](https://www.w3.org/TR/eventsource/)提供支持。 服务器发送的事件是相同的“HTTP流式传输”技术的另一种变体，除了从服务器推送的事件根据W3C服务器发送的事件规范格式化。

服务器发送的事件可用于其预期的目的，即将事件从服务器推送到客户端。 在Spring MVC中很容易，只需要返回一个`SseEmitter`类型的值。

请注意，Internet Explorer不支持服务器发送事件，而对于更高级的Web应用程序消息传递场景（如在线游戏，协作，财务应用程序等），最好考虑Spring的WebSocket支持，其中包括SockJS风格的WebSocket仿真回落到非常广泛的浏览器（包括Internet Explorer）以及更高级别的消息传递模式，用于通过更多以消息为中心的体系结构中的发布订阅模型与客户端进行交互。有关进一步的背景，请参阅[以下博文](http://blog.pivotal.io/pivotal/products/websocket-architecture-in-spring-4-0)。

#### HTTP直接流向OutputStream

`ResponseBodyEmitter`允许通过`HttpMessageConverter`将对象写入响应来发送事件。 这可能是最常见的情况，例如编写JSON数据时。 但是有时候绕过消息转换并直接写入响应`OutputStream`（例如文件下载）是有用的。 这可以通过`StreamingResponseBody`返回值类型来完成。

这是一个例子：

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

请注意，`StreamingResponseBody`也可以用作`ResponseEntity`中的主体，以便自定义响应的状态和标题。

#### 配置异步请求处理

##### Servlet容器配置

对于配置`web.xml`为确保更新到版本3.0的应用程序：

```java
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            http://java.sun.com/xml/ns/javaee
            http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
    version="3.0">

    ...

</web-app>
```

必须在`DispatcherServlet`上通过`web.xml`中的`<async-supported>true</async-supported>`子元素启用异步支持。 另外，任何参与异步请求处理的`Filter`都必须配置为支持ASYNC分派器类型。 为Spring Framework提供的所有过滤器启用ASYNC分派器类型应该是安全的，因为它们通常会扩展`OncePerRequestFilter`，并且运行时检查过滤器是否需要参与异步调度。

以下是一些web.xml配置示例：

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

如果通过`WebApplicationInitializer`使用Servlet 3，基于Java的配置，则还需要像`web.xml`一样设置“asyncSupported”标志以及ASYNC分派器类型。 为了简化所有这些配置，可以考虑扩展`AbstractDispatcherServletInitializer`，或者更好的`AbstractAnnotationConfigDispatcherServletInitializer`，它可以自动设置这些选项，并且可以很容易地注册`Filter`实例。

##### Spring MVC 配置

MVC Java配置和MVC命名空间提供了用于配置异步请求处理的选项。`WebMvcConfigurer`具有`configureAsyncSupport`方法，而`<mvc:annotation-driven>`具有`<async-support>`子元素。

这些允许你配置默认的超时值用于异步请求，如果没有设置则取决于底层的Servlet容器（例如Tomcat上的10秒）。 你也可以配置一个`AsyncTaskExecutor`来执行从控制器方法返回的`Callable`实例。 强烈建议配置此属性，因为默认情况下，Spring MVC使用`SimpleAsyncTaskExecutor`。 MVC Java配置和MVC命名空间还允许您注册`CallableProcessingInterceptor`和`DeferredResultProcessingInterceptor`实例。

如果您需要覆盖特定`DeferredResult`的默认超时值，可以使用适当的类构造函数来完成。 同样，对于`Callable`，可以将其包装在`WebAsyncTask`中，并使用适当的类构造函数来自定义超时值。 `WebAsyncTask`的类构造函数也允许提供一个`AsyncTaskExecutor`。

