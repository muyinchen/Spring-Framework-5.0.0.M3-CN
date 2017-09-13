### 18.16.13Advanced Customizations with MVC Java Config

As you can see from the above examples, MVC Java config and the MVC namespace provide higher level constructs that do not require deep knowledge of the underlying beans created for you. Instead it helps you to focus on your application needs. However, at some point you may need more fine-grained control or you may simply wish to understand the underlying configuration.

The first step towards more fine-grained control is to see the underlying beans created for you. In MVC Java config you can see the javadocs and the`@Bean`methods in`WebMvcConfigurationSupport`. The configuration in this class is automatically imported through the`@EnableWebMvc`annotation. In fact if you open`@EnableWebMvc`you can see the`@Import`statement.

The next step towards more fine-grained control is to customize a property on one of the beans created in`WebMvcConfigurationSupport`or perhaps to provide your own instance. This requires two things — remove the`@EnableWebMvc`annotation in order to prevent the import and then extend from`DelegatingWebMvcConfiguration`, a subclass of`WebMvcConfigurationSupport`. Here is an example:

```java
@Configuration
public class WebConfig extends DelegatingWebMvcConfiguration {

	@Override
	public void addInterceptors(InterceptorRegistry registry){
		// ...
	}

	@Override
	@Bean
	public RequestMappingHandlerAdapter requestMappingHandlerAdapter() {
		// Create or let "super" create the adapter
		// Then customize one of its properties
	}

}
```

| ![](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png) |
| :--- |
| An application should have only one configuration extending`DelegatingWebMvcConfiguration`or a single`@EnableWebMvc`annotated class, since they both register the same underlying beans.Modifying beans in this way does not prevent you from using any of the higher-level constructs shown earlier in this section.`WebMvcConfigurerAdapter`subclasses and`WebMvcConfigurer`implementations are still being used. |



