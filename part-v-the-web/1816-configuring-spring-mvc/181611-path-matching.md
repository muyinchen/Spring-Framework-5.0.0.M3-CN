### 18.16.11Path Matching

This allows customizing various settings related to URL mapping and path matching. For details on the individual options check out the[PathMatchConfigurer](http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/web/servlet/config/annotation/PathMatchConfigurer.html)API.

Below is an example in Java config:

```java
@Configuration
@EnableWebMvc
public class WebConfig extends WebMvcConfigurerAdapter {

	@Override
	public void configurePathMatch(PathMatchConfigurer configurer) {
		configurer
		    .setUseSuffixPatternMatch(true)
		    .setUseTrailingSlashMatch(false)
		    .setUseRegisteredSuffixPatternMatch(true)
		    .setPathMatcher(antPathMatcher())
		    .setUrlPathHelper(urlPathHelper());
	}

	@Bean
	public UrlPathHelper urlPathHelper() {
	    //...
	}

	@Bean
	public PathMatcher antPathMatcher() {
	    //...
	}

}
```

And the same in XML, use the `<mvc:path-matching>` element:

```java
<mvc:annotation-driven>
    <mvc:path-matching
        suffix-pattern="true"
        trailing-slash="false"
        registered-suffixes-only="true"
        path-helper="pathHelper"
        path-matcher="pathMatcher"/>
</mvc:annotation-driven>

<bean id="pathHelper" class="org.example.app.MyPathHelper"/>
<bean id="pathMatcher" class="org.example.app.MyPathMatcher"/>
```



