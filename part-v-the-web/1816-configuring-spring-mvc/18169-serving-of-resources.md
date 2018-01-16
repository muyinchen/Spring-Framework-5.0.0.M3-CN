### 18.16.9服务于资源

这个选项允许一个特定的URL模式之后的静态资源请求由一个`ResourceHttpRequestHandler`从任何一个资源位置列表中提供。 这提供了一种方便的方式来提供来自Web应用程序根目录以外的位置的静态资源，包括类路径上的位置。 `cache-period`属性可以用来设置将来过期头（1年是诸如Page Speed和YSlow等优化工具的推荐），以便客户更有效地使用它们。 该处理程序还可以正确评估`Last-Modified`标头（如果存在），以便适当地返回`304`状态码，避免客户端已经缓存的资源产生不必要的开销。 例如，要使用Web应用程序根目录中`public-resources`目录中的`/resources/**`的URL模式提供资源请求，可以使用：

```java
@Configuration
@EnableWebMvc
public class WebConfig extends WebMvcConfigurerAdapter {

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/resources/**").addResourceLocations("/public-resources/");
    }

}
```

和XML一样：

```java
<mvc:resources mapping="/resources/**" location="/public-resources/"/>
```

要为这些资源提供1年的将来期限，以确保最大限度地利用浏览器缓存并减少浏览器发出的HTTP请求：

```java
@Configuration
@EnableWebMvc
public class WebConfig extends WebMvcConfigurerAdapter {

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/resources/**").addResourceLocations("/public-resources/").setCachePeriod(31556926);
    }

}
```

在XML中：

```java
<mvc:resources mapping="/resources/**" location="/public-resources/" cache-period="31556926"/>
```

有关更多详细信息，请参阅[HTTP缓存对静态资源的支持](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-caching-static-resources)。

`mapping`属性必须是可由`SimpleUrlHandlerMapping`使用的Ant模式，而`location`属性必须指定一个或多个有效的资源目录位置。 可以使用逗号分隔的值列表来指定多个资源位置。 指定的位置将按照指定的顺序检查任何给定请求是否存在资源。 例如，要启用来自Web应用程序根目录和来自`/META-INF/public-web-resources/`的已知路径的资源，请在类路径的任何jar中使用：

```java
@EnableWebMvc
@Configuration
public class WebConfig extends WebMvcConfigurerAdapter {

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/resources/**")
                .addResourceLocations("/", "classpath:/META-INF/public-web-resources/");
    }

}
```

在XML中：

```java
<mvc:resources mapping="/resources/**" location="/, classpath:/META-INF/public-web-resources/"/>
```

当部署应用程序的新版本时可能会更改资源时，建议您将版本字符串并入用于请求资源的映射模式，以便您可以强制客户端请求新部署的应用程序资源版本。 对版本化URL的支持已内置到框架中，并且可以通过在资源处理程序上配置资源链来启用。 该链由多个`ResourceResolver`实例组成，后面紧跟着一个或多个`ResourceTransformer`实例。 他们一起可以提供任意的解决方案和资源转换。

内置的`VersionResourceResolver`可以配置不同的策略。 例如，`FixedVersionStrategy`可以使用属性，日期或其他作为版本。 `ContentVersionStrategy`使用从资源内容（称为“fingerprinting”URL）计算出的MD5哈希值。 请注意，在服务资源时，`VersionResourceResolver`将自动使用已解析的版本字符串作为HTTP ETag标头值。

`ContentVersionStrategy`是一个很好的默认选择，除非无法使用（例如使用JavaScript模块加载器）。 您可以针对不同的模式配置不同的版本策略，如下所示。 请记住，计算基于内容的版本是昂贵的，因此应该在生产中启用资源链高速缓存。

Java配置示例;

```java
@Configuration
@EnableWebMvc
public class WebConfig extends WebMvcConfigurerAdapter {

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/resources/**")
                .addResourceLocations("/public-resources/")
                .resourceChain(true).addResolver(
                    new VersionResourceResolver().addContentVersionStrategy("/**"));
    }

}
```

XML example:

```java
<mvc:resources mapping="/resources/**" location="/public-resources/">
    <mvc:resource-chain>
        <mvc:resource-cache/>
        <mvc:resolvers>
            <mvc:version-resolver>
                <mvc:content-version-strategy patterns="/**"/>
            </mvc:version-resolver>
        </mvc:resolvers>
    </mvc:resource-chain>
</mvc:resources>
```

为了上述工作，应用程序还必须呈现带有版本的URL。 最简单的方法是配置包装响应的`ResourceUrlEncodingFilter`，并覆盖`encodeURL`方法。 这将在JSP，FreeMarker和其他任何调用`encodeURL`方法的视图技术中起作用。 或者，应用程序也可以直接注入并使用`ResourceUrlProvider` bean，该bean是使用MVC Java配置和MVC命名空间自动声明的。

WebJars也支持`WebJarsResourceResolver`，当`"org.webjars:webjars-locator"`库位于类路径上时，会自动注册。 此解析器允许资源链从HTTP GET请求解析版本不可知的库`"GET /jquery/jquery.min.js"`将返回资源`"/jquery/1.2.0/jquery.min.js"`。 它还通过在模板→重写资源URL来工作。

