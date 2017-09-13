### 18.16.5Interceptors

You can configure`HandlerInterceptors`or`WebRequestInterceptors`to be applied to all incoming requests or restricted to specific URL path patterns.

An example of registering interceptors in Java:

```java
@Configuration
@EnableWebMvc
public class WebConfig extends WebMvcConfigurerAdapter {

	@Override
	public void addInterceptors(InterceptorRegistry registry) {
		registry.addInterceptor(new LocaleInterceptor());
		registry.addInterceptor(new ThemeInterceptor()).addPathPatterns("/**").excludePathPatterns("/admin/**");
		registry.addInterceptor(new SecurityInterceptor()).addPathPatterns("/secure/*");
	}

}
```

And in XML use the` <mvc:interceptors> `element:

```java
<mvc:interceptors>
	<bean class="org.springframework.web.servlet.i18n.LocaleChangeInterceptor"/>
	<mvc:interceptor>
		<mvc:mapping path="/**"/>
		<mvc:exclude-mapping path="/admin/**"/>
		<bean class="org.springframework.web.servlet.theme.ThemeChangeInterceptor"/>
	</mvc:interceptor>
	<mvc:interceptor>
		<mvc:mapping path="/secure/*"/>
		<bean class="org.example.SecurityInterceptor"/>
	</mvc:interceptor>
</mvc:interceptors>
```



