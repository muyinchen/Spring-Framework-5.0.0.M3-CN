## 18.7Building URIs

Spring MVC provides a mechanism for building and encoding a URI using`UriComponentsBuilder`and`UriComponents`.

For example you can expand and encode a URI template string:

```java
UriComponents uriComponents = UriComponentsBuilder.fromUriString(
		"http://example.com/hotels/{hotel}/bookings/{booking}").build();

URI uri = uriComponents.expand("42", "21").encode().toUri();
```

Note that`UriComponents`is immutable and the`expand()`and`encode()`operations return new instances if necessary.

You can also expand and encode using individual URI components:

```java
UriComponents uriComponents = UriComponentsBuilder.newInstance()
		.scheme("http").host("example.com").path("/hotels/{hotel}/bookings/{booking}").build()
		.expand("42", "21")
		.encode();
```

In a Servlet environment the`ServletUriComponentsBuilder`sub-class provides static factory methods to copy available URL information from a Servlet requests:

```java
HttpServletRequest request = ...

// Re-use host, scheme, port, path and query string
// Replace the "accountId" query param

ServletUriComponentsBuilder ucb = ServletUriComponentsBuilder.fromRequest(request)
		.replaceQueryParam("accountId", "{id}").build()
		.expand("123")
		.encode();
```

Alternatively, you may choose to copy a subset of the available information up to and including the context path:

```java
// Re-use host, port and context path
// Append "/accounts" to the path

ServletUriComponentsBuilder ucb = ServletUriComponentsBuilder.fromContextPath(request)
		.path("/accounts").build()
```

Or in cases where the`DispatcherServlet`is mapped by name \(e.g.`/main/*`\), you can also have the literal part of the servlet mapping included:

```java
// Re-use host, port, context path
// Append the literal part of the servlet mapping to the path
// Append "/accounts" to the path

ServletUriComponentsBuilder ucb = ServletUriComponentsBuilder.fromServletMapping(request)
		.path("/accounts").build()
```



