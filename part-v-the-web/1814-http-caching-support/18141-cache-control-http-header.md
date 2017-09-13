### 18.14.1Cache-Control HTTP header

Spring Web MVC supports many use cases and ways to configure "Cache-Control" headers for an application. While[RFC 7234 Section 5.2.2](https://tools.ietf.org/html/rfc7234#section-5.2.2)completely describes that header and its possible directives, there are several ways to address the most common cases.

Spring Web MVC uses a configuration convention in several of its APIs:`setCachePeriod(int seconds)`:

* A`-1`value won’t generate a`'Cache-Control'`response header.

* A`0`value will prevent caching using the`'Cache-Control: no-store'`directive.

* An`n > 0`value will cache the given response for`n`seconds using the`'Cache-Control: max-age=n'`directive.

The[`CacheControl`](http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/http/CacheControl.html)builder class simply describes the available "Cache-Control" directives and makes it easier to build your own HTTP caching strategy. Once built, a`CacheControl`instance can then be accepted as an argument in several Spring Web MVC APIs.

```java
// Cache for an hour - "Cache-Control: max-age=3600"
   CacheControl ccCacheOneHour = CacheControl.maxAge(1, TimeUnit.HOURS);

   // Prevent caching - "Cache-Control: no-store"
   CacheControl ccNoStore = CacheControl.noStore();

   // Cache for ten days in public and private caches,
   // public caches should not transform the response
   // "Cache-Control: max-age=864000, public, no-transform"
   CacheControl ccCustom = CacheControl.maxAge(10, TimeUnit.DAYS)
   									.noTransform().cachePublic();
```



