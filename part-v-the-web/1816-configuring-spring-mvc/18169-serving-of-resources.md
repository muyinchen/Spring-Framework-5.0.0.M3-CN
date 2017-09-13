### 18.16.9Serving of Resources

This option allows static resource requests following a particular URL pattern to be served by a`ResourceHttpRequestHandler`from any of a list of`Resource`locations. This provides a convenient way to serve static resources from locations other than the web application root, including locations on the classpath. The`cache-period`property may be used to set far future expiration headers \(1 year is the recommendation of optimization tools such as Page Speed and YSlow\) so that they will be more efficiently utilized by the client. The handler also properly evaluates the`Last-Modified`header \(if present\) so that a`304`status code will be returned as appropriate, avoiding unnecessary overhead for resources that are already cached by the client. For example, to serve resource requests with a URL pattern of`/resources/**`from a`public-resources`directory within the web application root you would use:

```java
@Configuration
@EnableWebMvc
public class WebConfig extends WebMvcConfigurerAdapter {

	@Override
	public void addResourceHandlers(ResourceHandlerRegistry registry) {
		registry.addResourceHandler("/resources/**").addResourceLocations("/public-resources/");
	}

}
```

And the same in XML:

```java
<mvc:resources mapping="/resources/**" location="/public-resources/"/>
```

To serve these resources with a 1-year future expiration to ensure maximum use of the browser cache and a reduction in HTTP requests made by the browser:

```java
@Configuration
@EnableWebMvc
public class WebConfig extends WebMvcConfigurerAdapter {

	@Override
	public void addResourceHandlers(ResourceHandlerRegistry registry) {
		registry.addResourceHandler("/resources/**").addResourceLocations("/public-resources/").setCachePeriod(31556926);
	}

}
```

And in XML:

```java
<mvc:resources mapping="/resources/**" location="/public-resources/" cache-period="31556926"/>
```

For more details, see[HTTP caching support for static resources](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-caching-static-resources).

The`mapping`attribute must be an Ant pattern that can be used by`SimpleUrlHandlerMapping`, and the`location`attribute must specify one or more valid resource directory locations. Multiple resource locations may be specified using a comma-separated list of values. The locations specified will be checked in the specified order for the presence of the resource for any given request. For example, to enable the serving of resources from both the web application root and from a known path of`/META-INF/public-web-resources/`in any jar on the classpath use:

```java
@EnableWebMvc
@Configuration
public class WebConfig extends WebMvcConfigurerAdapter {

	@Override
	public void addResourceHandlers(ResourceHandlerRegistry registry) {
		registry.addResourceHandler("/resources/**")
				.addResourceLocations("/", "classpath:/META-INF/public-web-resources/");
	}

}
```

And in XML:

```java
<mvc:resources mapping="/resources/**" location="/, classpath:/META-INF/public-web-resources/"/>
```

When serving resources that may change when a new version of the application is deployed it is recommended that you incorporate a version string into the mapping pattern used to request the resources so that you may force clients to request the newly deployed version of your application’s resources. Support for versioned URLs is built into the framework and can be enabled by configuring a resource chain on the resource handler. The chain consists of one more`ResourceResolver`instances followed by one or more`ResourceTransformer`instances. Together they can provide arbitrary resolution and transformation of resources.

The built-in`VersionResourceResolver`can be configured with different strategies. For example a`FixedVersionStrategy`can use a property, a date, or other as the version. A`ContentVersionStrategy`uses an MD5 hash computed from the content of the resource \(known as "fingerprinting" URLs\). Note that the`VersionResourceResolver`will automatically use the resolved version strings as HTTP ETag header values when serving resources.

`ContentVersionStrategy`is a good default choice to use except in cases where it cannot be used \(e.g. with JavaScript module loaders\). You can configure different version strategies against different patterns as shown below. Keep in mind also that computing content-based versions is expensive and therefore resource chain caching should be enabled in production.

Java config example;

```java
@Configuration
@EnableWebMvc
public class WebConfig extends WebMvcConfigurerAdapter {

	@Override
	public void addResourceHandlers(ResourceHandlerRegistry registry) {
		registry.addResourceHandler("/resources/**")
				.addResourceLocations("/public-resources/")
				.resourceChain(true).addResolver(
					new VersionResourceResolver().addContentVersionStrategy("/**"));
	}

}
```

XML example:

```java
<mvc:resources mapping="/resources/**" location="/public-resources/">
	<mvc:resource-chain>
		<mvc:resource-cache/>
		<mvc:resolvers>
			<mvc:version-resolver>
				<mvc:content-version-strategy patterns="/**"/>
			</mvc:version-resolver>
		</mvc:resolvers>
	</mvc:resource-chain>
</mvc:resources>
```

In order for the above to work the application must also render URLs with versions. The easiest way to do that is to configure the`ResourceUrlEncodingFilter`which wraps the response and overrides its`encodeURL`method. This will work in JSPs, FreeMarker, and any other view technology that calls the response`encodeURL`method. Alternatively, an application can also inject and use directly the`ResourceUrlProvider`bean, which is automatically declared with the MVC Java config and the MVC namespace.

Webjars are also supported with`WebJarsResourceResolver`, which is automatically registered when the`"org.webjars:webjars-locator"`library is on classpath. This resolver allows the resource chain to resolve version agnostic libraries from HTTP GET requests`"GET /jquery/jquery.min.js"`will return resource`"/jquery/1.2.0/jquery.min.js"`. It also works by rewriting resource URLs in templates`→`.

