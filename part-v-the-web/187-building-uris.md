## 18.7构建URI

Spring MVC提供了一种使用`UriComponentsBuilder`和`UriComponents`构建和编码URI的机制。

例如，您可以扩展和编码URI模板字符串：

```java
UriComponents uriComponents = UriComponentsBuilder.fromUriString(
        "http://example.com/hotels/{hotel}/bookings/{booking}").build();

URI uri = uriComponents.expand("42", "21").encode().toUri();
```

请注意，UriComponents是不可变的，如果需要，`expand()`和`encode()`操作将返回新的实例。

您还可以使用各个URI组件进行扩展和编码：

```java
UriComponents uriComponents = UriComponentsBuilder.newInstance()
        .scheme("http").host("example.com").path("/hotels/{hotel}/bookings/{booking}").build()
        .expand("42", "21")
        .encode();
```

在Servlet环境中，`ServletUriComponentsBuilder`子类提供静态工厂方法从Servlet请求中复制可用的URL信息：

```java
HttpServletRequest request = ...

// Re-use host, scheme, port, path and query string
// Replace the "accountId" query param

ServletUriComponentsBuilder ucb = ServletUriComponentsBuilder.fromRequest(request)
        .replaceQueryParam("accountId", "{id}").build()
        .expand("123")
        .encode();
```

或者，您可以选择将可用信息的一个子集复制到上下文路径（包括上下文路径）：

```java
// Re-use host, port and context path
// Append "/accounts" to the path

ServletUriComponentsBuilder ucb = ServletUriComponentsBuilder.fromContextPath(request)
        .path("/accounts").build()
```

或者在`DispatcherServlet`按名称映射（例如`/main/*`）的情况下，还可以包含servlet映射的文字部分：

```java
// Re-use host, port, context path
// Append the literal part of the servlet mapping to the path
// Append "/accounts" to the path

ServletUriComponentsBuilder ucb = ServletUriComponentsBuilder.fromServletMapping(request)
        .path("/accounts").build()
```



