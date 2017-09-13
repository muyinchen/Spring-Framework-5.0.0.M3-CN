### 18.16.12Message Converters

Customization of`HttpMessageConverter`can be achieved in Java config by overriding[`configureMessageConverters()`](http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/web/servlet/config/annotation/WebMvcConfigurerAdapter.html#configureMessageConverters-java.util.List-)if you want to replace the default converters created by Spring MVC, or by overriding[`extendMessageConverters()`](http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/web/servlet/config/annotation/WebMvcConfigurerAdapter.html#extendMessageConverters-java.util.List-)if you just want to customize them or add additional converters to the default ones.

Below is an example that adds Jackson JSON and XML converters with a customized`ObjectMapper`instead of default ones:

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

In this example,`Jackson2ObjectMapperBuilder`is used to create a common configuration for both`MappingJackson2HttpMessageConverter`and`MappingJackson2XmlHttpMessageConverter`with indentation enabled, a customized date format and the registration of[jackson-module-parameter-names](https://github.com/FasterXML/jackson-module-parameter-names)that adds support for accessing parameter names \(feature added in Java 8\).

| ![](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png) |
| :--- |
| Enabling indentation with Jackson XML support requires[`woodstox-core-asl`](https://search.maven.org/#search%7Cgav%7C1%7Cg%3A%22org.codehaus.woodstox%22%20AND%20a%3A%22woodstox-core-asl%22)dependency in addition to[`jackson-dataformat-xml`](https://search.maven.org/#search%7Cga%7C1%7Ca%3A%22jackson-dataformat-xml%22)one. |

Other interesting Jackson modules are available:

1. [jackson-datatype-money](https://github.com/zalando/jackson-datatype-money): support for`javax.money`types \(unofficial module\)

2. [jackson-datatype-hibernate](https://github.com/FasterXML/jackson-datatype-hibernate): support for Hibernate specific types and properties \(including lazy-loading aspects\)

It is also possible to do the same in XML:

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



