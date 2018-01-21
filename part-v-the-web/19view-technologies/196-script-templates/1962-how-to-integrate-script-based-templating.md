### 19.6.2How to integrate script based templating

To be able to use script templates, you have to configure it in order to specify various parameters like the script engine to use, the script files to load and what function should be called to render the templates. This is done thanks to a`ScriptTemplateConfigurer`bean and optional script files.

For example, in order to render Mustache templates thanks to the Nashorn Javascript engine provided with Java 8+, you should declare the following configuration:

```java
@Configuration
@EnableWebMvc
public class MustacheConfig extends WebMvcConfigurerAdapter {

	@Override
	public void configureViewResolvers(ViewResolverRegistry registry) {
		registry.scriptTemplate();
	}

	@Bean
	public ScriptTemplateConfigurer configurer() {
		ScriptTemplateConfigurer configurer = new ScriptTemplateConfigurer();
		configurer.setEngineName("nashorn");
		configurer.setScripts("mustache.js");
		configurer.setRenderObject("Mustache");
		configurer.setRenderFunction("render");
		return configurer;
	}
}
```

The XML counterpart using MVC namespace is:

```java
<mvc:annotation-driven/>

<mvc:view-resolvers>
	<mvc:script-template/>
</mvc:view-resolvers>

<mvc:script-template-configurer engine-name="nashorn" render-object="Mustache" render-function="render">
	<mvc:script location="mustache.js"/>
</mvc:script-template-configurer>
```

The controller is exactly what you should expect:

```java
@Controller
public class SampleController {

	@RequestMapping
	public ModelAndView test() {
		ModelAndView mav  = new ModelAndView();
		mav.addObject("title", "Sample title").addObject("body", "Sample body");
		mav.setViewName("template.html");
		return mav;
	}
}
```

And the Mustache template is:

```js
<html>
	<head>
		<title>{{title}}</title>
	</head>
	<body>
		<p>{{body}}</p>
	</body>
</html>
```

The render function is called with the following parameters:

* `String template`: the template content

* `Map model`: the view model

* `String url`: the template url \(since 4.2.2\)

`Mustache.render()`is natively compatible with this signature, so you can call it directly.

If your templating technology requires some customization, you may provide a script that implements a custom render function. For example,[Handlerbars](http://handlebarsjs.com/)needs to compile templates before using them, and requires a[polyfill](https://en.wikipedia.org/wiki/Polyfill)in order to emulate some browser facilities not available in the server-side script engine.

```java
@Configuration
@EnableWebMvc
public class MustacheConfig extends WebMvcConfigurerAdapter {

	@Override
	public void configureViewResolvers(ViewResolverRegistry registry) {
		registry.scriptTemplate();
	}

	@Bean
	public ScriptTemplateConfigurer configurer() {
		ScriptTemplateConfigurer configurer = new ScriptTemplateConfigurer();
		configurer.setEngineName("nashorn");
		configurer.setScripts("polyfill.js", "handlebars.js", "render.js");
		configurer.setRenderFunction("render");
		configurer.setSharedEngine(false);
		return configurer;
	}
}
```

| ![](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png) |
| :--- |
| Setting the`sharedEngine`property to`false`is required when using non thread-safe script engines with templating libraries not designed for concurrency, like Handlebars or React running on Nashorn for example. In that case, Java 8u60 or greater is required due to[this bug](https://bugs.openjdk.java.net/browse/JDK-8076099). |

`polyfill.js`only defines the`window`object needed by Handlebars to run properly:

```js
var window = {};
```

This basic`render.js`implementation compiles the template before using it. A production ready implementation should also store and reused cached templates / pre-compiled templates. This can be done on the script side, as well as any customization you need \(managing template engine configuration for example\).

```js
function render(template, model) {
	var compiledTemplate = Handlebars.compile(template);
	return compiledTemplate(model);
}
```

Check out Spring script templates unit tests \([java](https://github.com/spring-projects/spring-framework/tree/master/spring-webmvc/src/test/java/org/springframework/web/servlet/view/script),[resources](https://github.com/spring-projects/spring-framework/tree/master/spring-webmvc/src/test/resources/org/springframework/web/servlet/view/script)\) for more configuration examples.

