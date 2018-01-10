### 18.14.3支持控制器中的Cache-Control，ETag和Last-Modified响应头

控制器可以支持`'Cache-Control'`，`'ETag'`和/或`'If-Modified-Since'`HTTP请求。 如果在响应中设置`'Cache-Control'`头部，这确实是推荐的。 这包括计算给定请求的lastmodified `long`和/或Etag值，并将其与`'If-Modified-Since'`请求标头值进行比较，并可能返回状态码304（未修改）的响应。

如[“使用HttpEntity”一节所述](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-ann-httpentity)，控制器可以使用`HttpEntity`类型与请求/响应进行交互。 返回`ResponseEntity`的控制器可以包含HTTP缓存信息，如下所示：

```java
@GetMapping("/book/{id}")
public ResponseEntity<Book> showBook(@PathVariable Long id) {

    Book book = findBook(id);
    String version = book.getVersion();

    return ResponseEntity
                .ok()
                .cacheControl(CacheControl.maxAge(30, TimeUnit.DAYS))
                .eTag(version) // lastModified is also available
                .body(book);
}
```

如果客户端发送的条件标头与控制器设置的缓存信息匹配，这样做将不仅包括响应中的头`'ETag'`和`'Cache-Control'`头，**还会将响应转换`HTTP 304 Not Modified`为空体**。

`@RequestMapping`方法也可能希望支持相同的行为。 这可以实现如下：

```java
@RequestMapping
public String myHandleMethod(WebRequest webRequest, Model model) {

    long lastModified = // 1. 应用程序特定的计算

    if (request.checkNotModified(lastModified)) {
        // 2. 快捷方式退出 - 无需进一步处理必需
        return null;
    }

    // 3. 或另外请求处理，实际准备内容
    model.addAttribute(...);
    return "myViewName";
}
```

这里有两个关键元素：调用`request.checkNotModified(lastModified)`并返回`null`。 前者在返回`true`之前设置适当的响应状态和标题。 后者与前者结合使得Spring MVC不再对请求做进一步的处理。

请注意，有3种变体：

* `request.checkNotModified(lastModified)`将lastModified与`'If-Modified-Since'`或`'If-Unmodified-Since'`请求头进行比较

* `request.checkNotModified(eTag)`将eTag与`'If-None-Match'`请求标头进行比较

* `request.checkNotModified(eTag, lastModified)`同时具有这两个条件，这意味着两个条件都应该是有效的

当接收到有条件的`'GET'`/`'HEAD'`请求时，`checkNotModified`将检查资源是否未被修改，如果是的话，将导致`HTTP 304 Not Modified`响应。 如果出现`'POST'`/`'PUT'`/`'DELETE'`请求，`checkNotModified`将检查资源是否未被修改，如果已经被修改，将导致`HTTP 409 Precondition Failed`响应阻止并发修改。

