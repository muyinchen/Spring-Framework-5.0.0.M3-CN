### 18.16.2Customizing the Provided Configuration

To customize the default configuration in Java you simply implement the`WebMvcConfigurer`interface or more likely extend the class`WebMvcConfigurerAdapter`and override the methods you need:

```java
@Configuration
@EnableWebMvc
public class WebConfig extends WebMvcConfigurerAdapter {

	// Override configuration methods...

}
```

To customize the default configuration of\`\`check what attributes and sub-elements it supports. You can view the[Spring MVC XML schema](http://schema.spring.io/mvc/spring-mvc.xsd)or use the code completion feature of your IDE to discover what attributes and sub-elements are available.

  


