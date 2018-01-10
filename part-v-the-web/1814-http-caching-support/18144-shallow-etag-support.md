### 18.14.4浅层ETag支持

ETags的支持由Servlet过滤器`ShallowEtagHeaderFilter`提供。 这是一个简单的Servlet过滤器，因此可以与任何Web框架结合使用。 `ShallowEtagHeaderFilter`过滤器创建所谓的浅ETag（相对于深ETag，稍后会详细介绍）。过滤器缓存呈现的JSP（或其他内容）的内容，生成一个MD5哈希，然后将其作为ETag 响应头。 下一次客户端发送相同资源的请求时，它将使用该散列作为`If-None-Match`值。 过滤器检测到这一点，再次呈现视图，并比较两个散列。 如果相等，则返回`304`。

请注意，此策略可节省网络带宽而不是CPU，因为必须为每个请求计算完整响应。 控制器级别的其他策略（如上所述）可以节省网络带宽并避免计算。

此过滤器具有`awriteETagag`参数，该参数配置过滤器写入弱ETag，如下所示：`W/“02a2d595e6ed9a0b24f027f2b63b134d6”`，如[RFC 7232第2.3节](https://tools.ietf.org/html/rfc7232#section-2.3)中所定义。

在`web.xml`中配置`ShallowEtagHeaderFilter`：

```java
<filter>
    <filter-name>etagFilter</filter-name>
    <filter-class>org.springframework.web.filter.ShallowEtagHeaderFilter</filter-class>
    <!-- 配置过滤器写入弱ETag的可选参数
    <init-param>
           <param-name>writeWeakETag</param-name>
           <param-value>true</param-value>
    </init-param>
    -->
</filter>

<filter-mapping>
    <filter-name>etagFilter</filter-name>
    <servlet-name>petclinic</servlet-name>
</filter-mapping>
```

或者在Servlet 3.0+环境中，

```java
public class MyWebAppInitializer extends AbstractDispatcherServletInitializer {

    // ...

    @Override
    protected Filter[] getServletFilters() {
        return new Filter[] { new ShallowEtagHeaderFilter() };
    }

}
```

有关详细信息，请参见[第18.15节“基于代码的Servlet容器初始化”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-container-config)。

