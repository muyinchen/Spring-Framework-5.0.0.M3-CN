### 18.16.8View Resolvers

The MVC config simplifies the registration of view resolvers.

The following is a Java config example that configures content negotiation view resolution using FreeMarker HTML templates and Jackson as a default`View`for JSON rendering:

```java
@Configuration
@EnableWebMvc
public class WebConfig extends WebMvcConfigurerAdapter {

	@Override
	public void configureViewResolvers(ViewResolverRegistry registry) {
		registry.enableContentNegotiation(new MappingJackson2JsonView());
		registry.jsp();
	}

}
```

And the same in XML:

```java
<mvc:view-resolvers>
	<mvc:content-negotiation>
		<mvc:default-views>
			<bean class="org.springframework.web.servlet.view.json.MappingJackson2JsonView"/>
		</mvc:default-views>
	</mvc:content-negotiation>
	<mvc:jsp/>
</mvc:view-resolvers>
```

Note however that FreeMarker, Tiles, Groovy Markup and script templates also require configuration of the underlying view technology.

The MVC namespace provides dedicated elements. For example with FreeMarker:

```java
<mvc:view-resolvers>
	<mvc:content-negotiation>
		<mvc:default-views>
			<bean class="org.springframework.web.servlet.view.json.MappingJackson2JsonView"/>
		</mvc:default-views>
	</mvc:content-negotiation>
	<mvc:freemarker cache="false"/>
</mvc:view-resolvers>

<mvc:freemarker-configurer>
	<mvc:template-loader-path location="/freemarker"/>
</mvc:freemarker-configurer>
```

In Java config simply add the respective "Configurer" bean:

```java
@Configuration
@EnableWebMvc
public class WebConfig extends WebMvcConfigurerAdapter {

	@Override
	public void configureViewResolvers(ViewResolverRegistry registry) {
		registry.enableContentNegotiation(new MappingJackson2JsonView());
		registry.freeMarker().cache(false);
	}

	@Bean
	public FreeMarkerConfigurer freeMarkerConfigurer() {
		FreeMarkerConfigurer configurer = new FreeMarkerConfigurer();
		configurer.setTemplateLoaderPath("/WEB-INF/");
		return configurer;
	}

}
```



