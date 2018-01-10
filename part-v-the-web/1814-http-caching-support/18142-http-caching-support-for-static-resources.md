### 18.14.2HTTP缓存支持静态资源

静态资源应该使用app4 ropriate的`'Cache-Control'`和条件头来达到最佳性能。[配置一个`ResourceHttpRequestHandler`](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-config-static-resources)来为静态资源提供服务，不仅通过读取文件的元数据本地写入`'Last-Modified'`头，而且还可以正确配置`'Cache-Control'` 头文件。

您可以在`ResourceHttpRequestHandler`上设置`cachePeriod`属性，也可以使用支持更多特定指令的`CacheControl`实例：

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

在XML中：

```java
<mvc:resources mapping="/resources/**" location="/public-resources/">
    <mvc:cache-control max-age="3600" cache-public="true"/>
</mvc:resources>
```



