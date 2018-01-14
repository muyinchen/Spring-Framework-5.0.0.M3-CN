### 18.16.6内容谈判

您可以配置Spring MVC如何根据请求确定请求的媒体类型。 可用的选项是检查文件扩展名的URL路径，检查“Accept”头，特定的查询参数，或者在没有请求时返回默认的内容类型。 默认情况下，首先检查请求URI中的路径扩展，然后检查“Accept”标头。

MVC Java配置和MVC命名空间默认情况下注册json，xml，rss，atom，如果相应的依赖关系在类路径上。 额外的路径扩展到媒体类型的映射也可以被明确注册，并且为了RFD攻击检测的目的（参见[“后缀模式匹配和RFD”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-ann-requestmapping-rfd)部分以获得更多细节）也具有将它们白名单作为安全扩展的效果。

以下是通过MVC Java配置自定义内容协商选项的示例：

```java
@Configuration
@EnableWebMvc
public class WebConfig extends WebMvcConfigurerAdapter {

    @Override
    public void configureContentNegotiation(ContentNegotiationConfigurer configurer) {
        configurer.mediaType("json", MediaType.APPLICATION_JSON);
    }
}
```

在MVC名称空间中，`<mvc:annotation-driven>`元素具有`content-negotiation-manager`属性，该属性需要一个`ContentNegotiationManager`，而`ContentNegotiationManager`又可以使用`ContentNegotiationManagerFactoryBean`创建：

```java
<mvc:annotation-driven content-negotiation-manager="contentNegotiationManager"/>

<bean id="contentNegotiationManager" class="org.springframework.web.accept.ContentNegotiationManagerFactoryBean">
    <property name="mediaTypes">
        <value>
            json=application/json
            xml=application/xml
        </value>
    </property>
</bean>
```

如果不使用MVC Java配置或MVC命名空间，则需要创建`ContentNegotiationManager`的一个实例，并使用它来配置`RequestMappingHandlerMapping`以实现请求映射，`RequestMappingHandlerAdapter`和`ExceptionHandlerExceptionResolver`实现内容协商。

请注意，`ContentNegotiatingViewResolver`现在也可以使用`ContentNegotiationManager`进行配置，因此您可以在整个Spring MVC中使用一个共享实例。

在更高级的情况下，配置多个`ContentNegotiationManager`实例可能会很有用，而这些实例又可能包含自定义的`ContentNegotiationStrategy`实现。 例如，你可以配置`ExceptionHandlerExceptionResolver`和一个`ContentNegotiationManager`，它始终将请求的媒体类型解析为`"application/json"`。 或者，如果没有请求内容类型，您可能需要插入具有逻辑的自定义策略来选择默认内容类型（例如XML或JSON）。

