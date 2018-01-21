### 19.3.1Configuration

Configuring the Groovy Markup Template Engine is quite easy:

```java
@Configuration
@EnableWebMvc
public class WebConfig extends WebMvcConfigurerAdapter {

       @Override
	public void configureViewResolvers(ViewResolverRegistry registry) {
		registry.groovy();
	}

	@Bean
	public GroovyMarkupConfigurer groovyMarkupConfigurer() {
		GroovyMarkupConfigurer configurer = new GroovyMarkupConfigurer();
		configurer.setResourceLoaderPath("/WEB-INF/");
		return configurer;
	}
}
```

The XML counterpart using the MVC namespace is:

```java
<mvc:annotation-driven/>

<mvc:view-resolvers>
	<mvc:groovy/>
</mvc:view-resolvers>

<mvc:groovy-configurer resource-loader-path="/WEB-INF/"/>
```



