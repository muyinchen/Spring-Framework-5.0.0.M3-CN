### 18.16.1Enabling the MVC Java Config or the MVC XML Namespace

To enable MVC Java config add the annotation`@EnableWebMvc`to one of your`@Configuration`classes:

```java
@Configuration
@EnableWebMvc
public class WebConfig {

}
```

To achieve the same in XML use the`mvc:annotation-driven`element in your DispatcherServlet context \(or in your root context if you have no DispatcherServlet context defined\):

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

The above registers a`RequestMappingHandlerMapping`, a`RequestMappingHandlerAdapter`, and an`ExceptionHandlerExceptionResolver`\(among others\) in support of processing requests with annotated controller methods using annotations such as`@RequestMapping`,`@ExceptionHandler`, and others.

It also enables the following:

1. Spring 3 style type conversion through a[ConversionService](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/validation.html#core-convert)instance in addition to the JavaBeans PropertyEditors used for Data Binding.

2. Support for[formatting](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/validation.html#format)Number fields using the`@NumberFormat`annotation through the`ConversionService`.

3. Support for[formatting](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/validation.html#format)`Date`,`Calendar`,`Long`, and Joda Time fields using the`@DateTimeFormat`annotation.

4. Support for[validating](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-config-validation)`@Controller`inputs with`@Valid`, if a JSR-303 Provider is present on the classpath.

5. `HttpMessageConverter`support for`@RequestBody`method parameters and`@ResponseBody`method return values from`@RequestMapping`or`@ExceptionHandler`methods.

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



