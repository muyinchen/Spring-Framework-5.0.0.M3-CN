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

   这是由mvc设置的HttpMessageConverters的完整列表：annotation-driven：

   1. `ByteArrayHttpMessageConverter`转换字节数组。

   2. `StringHttpMessageConverter`转换字符串。

   3. `ResourceHttpMessageConverter`为所有媒体类型转换为`org.springframework.core.io.Resource`。

   4. `SourceHttpMessageConverter转换为/从一个javax.xml.transform.Source`。

   5. `FormHttpMessageConverter`将表单数据转换为`MultiValueMap`或从`MultiValueMap`转换而来。

   6. `Jaxb2RootElementHttpMessageConverter`将Java对象转换为/从XML转换 - 如果存在JAXB2并且类路径中不存在Jackson 2 XML扩展，则添加它。

   7. `MappingJackson2HttpMessageConverter`转换为/从JSON - 如果Jackson 2存在于类路径中，则添加。

   8. `MappingJackson2XmlHttpMessageConverter`转换为/从XML - 如果[Jackson 2 XML扩展](https://github.com/FasterXML/jackson-dataformat-xml)存在于类路径中，则添加。

   9. `MappingJackson2SmileHttpMessageConverter`converts to/from Smile \(binary JSON\) — added if[Jackson 2 Smile extension](https://github.com/FasterXML/jackson-dataformats-binary/tree/master/smile)is present on the classpath.

   10.  `MappingJackson2SmileHttpMessageConverter`转换为/从Smile（二进制JSON） - 添加如果[Jackson 2 CBOR extension](https://github.com/FasterXML/jackson-dataformats-binary/tree/master/cbor)存在于类路径。

   11. `AtomFeedHttpMessageConverter`转换Atom提要 - 如果Rome出现在类路径中，则添加Atom提要。

   12. `RssChannelHttpMessageConverter`转换RSS源 - 如果Rome出现在类路径中，则添加。

   有关如何定制这些默认转换器的更多信息，请参见[第18.16.12节“消息转换器”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-config-message-converters)。

| ![](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png) |
| :--- |
|  Jackson JSON和XML转换器是使用[`Jackson2ObjectMapperBuilder`](http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/http/converter/json/Jackson2ObjectMapperBuilder.html)创建的`ObjectMapper`实例创建的，以便提供更好的默认配置。此构建器使用以下几种方法自定义Jackson的默认属性：[`DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES`](https://fasterxml.github.io/jackson-databind/javadoc/2.6/com/fasterxml/jackson/databind/DeserializationFeature.html#FAIL_ON_UNKNOWN_PROPERTIES)被禁用。 [`MapperFeature.DEFAULT_VIEW_INCLUSION`](https://fasterxml.github.io/jackson-databind/javadoc/2.6/com/fasterxml/jackson/databind/MapperFeature.html#DEFAULT_VIEW_INCLUSION)被禁用。如果在类路径中检测到以下模块，它将自动注册下列已知模块：[jackson-datatype-jdk7](https://github.com/FasterXML/jackson-datatype-jdk7)：支持`java.nio.file.Path`等Java 7类型。[jackson-datatype-joda](https://github.com/FasterXML/jackson-datatype-joda)：支持Joda-Time类型。[jackson-datatype-jsr310](https://github.com/FasterXML/jackson-datatype-jsr310)：支持Java 8 Date＆Time API类型。[jackson-datatype-jdk8](https://github.com/FasterXML/jackson-datatype-jdk8)：支持其他Java 8类型，如`Optional`。 |



