### 18.14.2HTTP caching support for static resources

Static resources should be served with appropriate`'Cache-Control'`and conditional headers for optimal performance.[Configuring a`ResourceHttpRequestHandler`](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-config-static-resources)for serving static resources not only natively writes`'Last-Modified'`headers by reading a file’s metadata, but also`'Cache-Control'`headers if properly configured.

You can set the`cachePeriod`attribute on a`ResourceHttpRequestHandler`or use a`CacheControl`instance, which supports more specific directives:

```java
@Configuration
@EnableWebMvc
public class WebConfig extends WebMvcConfigurerAdapter {

	@Override
	public void addResourceHandlers(ResourceHandlerRegistry registry) {
		registry.addResourceHandler("/resources/**")
				.addResourceLocations("/public-resources/")
				.setCacheControl(CacheControl.maxAge(1, TimeUnit.HOURS).cachePublic());
	}

}
```

And in XML:

```java
<mvc:resources mapping="/resources/**" location="/public-resources/">
	<mvc:cache-control max-age="3600" cache-public="true"/>
</mvc:resources>
```



