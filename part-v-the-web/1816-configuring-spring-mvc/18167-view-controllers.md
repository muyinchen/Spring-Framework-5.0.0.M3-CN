### 18.16.7View Controllers

This is a shortcut for defining a`ParameterizableViewController`that immediately forwards to a view when invoked. Use it in static cases when there is no Java controller logic to execute before the view generates the response.

An example of forwarding a request for`"/"`to a view called`"home"`in Java:

```java
@Configuration
@EnableWebMvc
public class WebConfig extends WebMvcConfigurerAdapter {

	@Override
	public void addViewControllers(ViewControllerRegistry registry) {
		registry.addViewController("/").setViewName("home");
	}

}
```

And the same in XML use the `<mvc:view-controller>` element:

```java
<mvc:view-controller path="/" view-name="home"/>
```



