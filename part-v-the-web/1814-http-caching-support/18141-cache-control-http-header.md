### 18.14.1缓存控制HTTP头

Spring Web MVC支持许多用例和为应用程序配置“Cache-Control”头的方法。 虽然[RFC 7234第5.2.2节](#)完整地描述了该头文件及其可能的指令，但有几种方法可以解决最常见的情况。

Spring Web MVC在其几个API中使用了一个配置约定：`setCachePeriod(int seconds)`：

* `-1`值不会生成`'Cache-Control'`响应头。

* 使用`'Cache-Control: no-store'`指令时，`0`值将会阻止缓存。

* 一个`n > 0`值将使用`'Cache-Control: max-age=n'`指令缓存给定的响应秒数。

[`CacheControl`](http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/http/CacheControl.html)构建器类简单地描述了可用的“Cache-Control”指令，使得构建自己的HTTP缓存策略更加容易。 一旦构建完成，一个`CacheControl`实例可以被接受为几个Spring Web MVC API中的一个参数。

```java
// 缓存一小时 - "Cache-Control: max-age=3600"
   CacheControl ccCacheOneHour = CacheControl.maxAge(1, TimeUnit.HOURS);

   // 防止缓存 - "Cache-Control: no-store"
   CacheControl ccNoStore = CacheControl.noStore();

   // 在公共和私有缓存中缓存十天,
   // 公共缓存不应该转换响应
   // "Cache-Control: max-age=864000, public, no-transform"
   CacheControl ccCustom = CacheControl.maxAge(10, TimeUnit.DAYS)
                                       .noTransform().cachePublic();
```



