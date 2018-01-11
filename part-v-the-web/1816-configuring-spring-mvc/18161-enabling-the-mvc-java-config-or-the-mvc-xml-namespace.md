### 18.16.1启用MVC Java Config或MVC XML命名空间

要启用MVC Java配置，请将注解`@EnableWebMvc`添加到其中一个`@Configuration`类中：

```java
@Configuration
@EnableWebMvc
public class WebConfig {

}
```

要在XML中实现相同的效果，请在DispatcherServlet上下文中使用`mvc:annotation-driven`元素（如果没有定义DispatcherServlet上下文，则在根上下文中）：

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:mvc="http://www.springframework.org/schema/mvc"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/mvc
        http://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <mvc:annotation-driven/>

</beans>
```

上面注册了一个`RequestMappingHandlerMapping`，一个`RequestMappingHandlerAdapter`和一个`ExceptionHandlerExceptionResolver`（以及其他），以支持使用注解的控制器方法（如`@RequestMapping`，`@ExceptionHandler`等）处理请求。

它还支持以下功能：

1. 除了用于数据绑定的JavaBean PropertyEditor之外，还通过[ConversionService](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/validation.html#core-convert)实例进行Spring 3样式类型转换。

2. 支持通过`ConversionService`使用`@NumberFormat`注解[格式](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/validation.html#format)数字字段。

3. 支持使用`@DateTimeFormat`注解[格式](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/validation.html#format)化`Date`,`Calendar`,`Long`和Joda时间字段。

4. 如果JSR-303提供程序存在于类路径中，则支持[validating](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-config-validation)使用`@Valid`验证`@Controller`输入。

5. `HttpMessageConverter`支持`@RequestBody`方法参数和`@ResponseBody`方法从`@RequestMapping`或`@ExceptionHandler`方法返回值。

This is the complete list of HttpMessageConverters set up by mvc:annotation-driven:

1. `ByteArrayHttpMessageConverter`converts byte arrays.

2. `StringHttpMessageConverter`converts strings.

3. `ResourceHttpMessageConverter`converts to/from`org.springframework.core.io.Resource`for all media types.

4. `SourceHttpMessageConverter`converts to/from a`javax.xml.transform.Source`.

5. `FormHttpMessageConverter`converts form data to/from a`MultiValueMap`.

6. `Jaxb2RootElementHttpMessageConverter`converts Java objects to/from XML — added if JAXB2 is present and Jackson 2 XML extension is not present on the classpath.

7. `MappingJackson2HttpMessageConverter`converts to/from JSON — added if Jackson 2 is present on the classpath.

8. `MappingJackson2XmlHttpMessageConverter`converts to/from XML — added if[Jackson 2 XML extension](https://github.com/FasterXML/jackson-dataformat-xml)is present on the classpath.

9. `MappingJackson2SmileHttpMessageConverter`converts to/from Smile \(binary JSON\) — added if[Jackson 2 Smile extension](https://github.com/FasterXML/jackson-dataformats-binary/tree/master/smile)is present on the classpath.

10. `MappingJackson2CborHttpMessageConverter`converts to/from CBOR — added if[Jackson 2 CBOR extension](https://github.com/FasterXML/jackson-dataformats-binary/tree/master/cbor)is present on the classpath.

11. `AtomFeedHttpMessageConverter`converts Atom feeds — added if Rome is present on the classpath.

12. `RssChannelHttpMessageConverter`converts RSS feeds — added if Rome is present on the classpath.

See[Section18.16.12, “Message Converters”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-config-message-converters)for more information about how to customize these default converters.

| ![](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png) |
| :--- |
| Jackson JSON and XML converters are created using`ObjectMapper`instances created by[`Jackson2ObjectMapperBuilder`](http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/http/converter/json/Jackson2ObjectMapperBuilder.html)in order to provide a better default configuration.This builder customizes Jackson’s default properties with the following ones:[`DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES`](https://fasterxml.github.io/jackson-databind/javadoc/2.6/com/fasterxml/jackson/databind/DeserializationFeature.html#FAIL_ON_UNKNOWN_PROPERTIES)is disabled.[`MapperFeature.DEFAULT_VIEW_INCLUSION`](https://fasterxml.github.io/jackson-databind/javadoc/2.6/com/fasterxml/jackson/databind/MapperFeature.html#DEFAULT_VIEW_INCLUSION)is disabled.It also automatically registers the following well-known modules if they are detected on the classpath:[jackson-datatype-jdk7](https://github.com/FasterXML/jackson-datatype-jdk7): support for Java 7 types like`java.nio.file.Path`.[jackson-datatype-joda](https://github.com/FasterXML/jackson-datatype-joda): support for Joda-Time types.[jackson-datatype-jsr310](https://github.com/FasterXML/jackson-datatype-jsr310): support for Java 8 Date & Time API types.[jackson-datatype-jdk8](https://github.com/FasterXML/jackson-datatype-jdk8): support for other Java 8 types like`Optional`. |



