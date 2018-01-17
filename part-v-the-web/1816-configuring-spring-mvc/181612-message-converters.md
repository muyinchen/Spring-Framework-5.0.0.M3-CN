### 18.16.12消息转换器

`HttpMessageConverter`的定制可以在Java配置中通过覆盖[`configureMessageConverters()`](http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/web/servlet/config/annotation/WebMvcConfigurerAdapter.html#configureMessageConverters-java.util.List-)来实现，如果你想替换Spring MVC创建的默认转换器，或者如果你只是想定制它们或者添加额外的转换器到默认转换器，可以重写[`extendMessageConverters()`](http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/web/servlet/config/annotation/WebMvcConfigurerAdapter.html#extendMessageConverters-java.util.List-)。

下面是一个例子，添加了Jackson JSON和XML转换器的自定义`ObjectMapper`而不是默认的：

```java
@Configuration
@EnableWebMvc
public class WebConfiguration extends WebMvcConfigurerAdapter {

    @Override
    public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
        Jackson2ObjectMapperBuilder builder = new Jackson2ObjectMapperBuilder()
                .indentOutput(true)
                .dateFormat(new SimpleDateFormat("yyyy-MM-dd"))
                .modulesToInstall(new ParameterNamesModule());
        converters.add(new MappingJackson2HttpMessageConverter(builder.build()));
        converters.add(new MappingJackson2XmlHttpMessageConverter(builder.xml().build()));
    }

}
```

在这个例子中，`Jackson2ObjectMapperBuilder`用于为`MappingJackson2HttpMessageConverter`和`MappingJackson2XmlHttpMessageConverter`创建通用配置，并启用缩进功能，自定义日期格式以及添加对访问参数名称的支持的[jackson-module-parameter-names](https://github.com/FasterXML/jackson-module-parameter-names)注册（Java 8中添加的功能）。

| ![](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png) |
| :--- |
| 使用Jackson XML支持启用缩进除了[`jackson-dataformat-xml`](https://search.maven.org/#search|ga|1|a%3A"jackson-dataformat-xml")之外，还需要[`woodstox-core-asl`](https://search.maven.org/#search|gav|1|g%3A"org.codehaus.woodstox" AND a%3A"woodstox-core-asl")依赖。 |

其他有趣的Jackson模块可用：

1. [jackson-datatype-money](https://github.com/zalando/jackson-datatype-money): 支持`javax.money`类型（非官方模块）

2. [jackson-datatype-hibernate](https://github.com/FasterXML/jackson-datatype-hibernate): 支持Hibernate的特定类型和属性（包括延迟加载方面）

在XML中也可以这样做：

```java
<mvc:annotation-driven>
    <mvc:message-converters>
        <bean class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter">
            <property name="objectMapper" ref="objectMapper"/>
        </bean>
        <bean class="org.springframework.http.converter.xml.MappingJackson2XmlHttpMessageConverter">
            <property name="objectMapper" ref="xmlMapper"/>
        </bean>
    </mvc:message-converters>
</mvc:annotation-driven>

<bean id="objectMapper" class="org.springframework.http.converter.json.Jackson2ObjectMapperFactoryBean"
      p:indentOutput="true"
      p:simpleDateFormat="yyyy-MM-dd"
      p:modulesToInstall="com.fasterxml.jackson.module.paramnames.ParameterNamesModule"/>

<bean id="xmlMapper" parent="objectMapper" p:createXmlMapper="true"/>
```



