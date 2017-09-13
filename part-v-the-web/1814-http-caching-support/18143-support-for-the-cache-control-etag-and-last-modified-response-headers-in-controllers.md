### 18.14.3Support for the Cache-Control, ETag and Last-Modified response headers in Controllers

Controllers can support`'Cache-Control'`,`'ETag'`, and/or`'If-Modified-Since'`HTTP requests; this is indeed recommended if a`'Cache-Control'`header is to be set on the response. This involves calculating a lastModified`long`and/or an Etag value for a given request, comparing it against the`'If-Modified-Since'`request header value, and potentially returning a response with status code 304 \(Not Modified\).

As described in[the section called “Using HttpEntity”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-ann-httpentity), controllers can interact with the request/response using`HttpEntity`types. Controllers returning`ResponseEntity`can include HTTP caching information in responses like this:

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

Doing this will not only include`'ETag'`and`'Cache-Control'`headers in the response, it will**also convert the response to anHTTP 304 Not Modifiedresponse with an empty body**if the conditional headers sent by the client match the caching information set by the Controller.

An`@RequestMapping`method may also wish to support the same behavior. This can be achieved as follows:

```java
@RequestMapping
public String myHandleMethod(WebRequest webRequest, Model model) {

	long lastModified = // 1. application-specific calculation

	if (request.checkNotModified(lastModified)) {
		// 2. shortcut exit - no further processing necessary
		return null;
	}

	// 3. or otherwise further request processing, actually preparing content
	model.addAttribute(...);
	return "myViewName";
}
```

There are two key elements here: calling`request.checkNotModified(lastModified)`and returning`null`. The former sets the appropriate response status and headers before it returns`true`. The latter, in combination with the former, causes Spring MVC to do no further processing of the request.

Note that there are 3 variants for this:

* `request.checkNotModified(lastModified)`compares lastModified with the`'If-Modified-Since'`or`'If-Unmodified-Since'`request header

* `request.checkNotModified(eTag)`compares eTag with the`'If-None-Match'`request header

* `request.checkNotModified(eTag, lastModified)`does both, meaning that both conditions should be valid

When receiving conditional`'GET'`/`'HEAD'`requests,`checkNotModified`will check that the resource has not been modified and if so, it will result in a`HTTP 304 Not Modified`response. In case of conditional`'POST'`/`'PUT'`/`'DELETE'`requests,`checkNotModified`will check that the resource has not been modified and if it has been, it will result in a`HTTP 409 Precondition Failed`response to prevent concurrent modifications.

