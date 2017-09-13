### 18.16.4Validation

Spring provides a[Validator interface](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/validation.html#validator)that can be used for validation in all layers of an application. In Spring MVC you can configure it for use as a global`Validator`instance, to be used whenever an`@Valid`or`@Validated`controller method argument is encountered, and/or as a local`Validator`within a controller through an`@InitBinder`method. Global and local validator instances can be combined to provide composite validation.

Spring also[supports JSR-303/JSR-349](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/validation.html#validation-beanvalidation-overview)Bean Validation via`LocalValidatorFactoryBean`which adapts the Spring`org.springframework.validation.Validator`interface to the Bean Validation`javax.validation.Validator`contract. This class can be plugged into Spring MVC as a global validator as described next.

By default use of`@EnableWebMvc`or```automatically registers Bean Validation support in Spring MVC through the``LocalValidatorFactoryBean\`when a Bean Validation provider such as Hibernate Validator is detected on the classpath.

| ![](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png) |
| :--- |
| Sometimes it’s convenient to have a`LocalValidatorFactoryBean`injected into a controller or another class. The easiest way to do that is to declare your own`@Bean`and also mark it with`@Primary`in order to avoid a conflict with the one provided with the MVC Java config.If you prefer to use the one from the MVC Java config, you’ll need to override the`mvcValidator`method from`WebMvcConfigurationSupport`and declare the method to explicitly return`LocalValidatorFactory`rather than`Validator`. See[Section18.16.13, “Advanced Customizations with MVC Java Config”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-config-advanced-java)for information on how to switch to extend the provided configuration. |

Alternatively you can configure your own global`Validator`instance:

```java
@Configuration
@EnableWebMvc
public class WebConfig extends WebMvcConfigurerAdapter {

	@Override
	public Validator getValidator(); {
		// return "global" validator
	}

}
```

and in XML:

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:mvc="http://www.springframework.org/schema/mvc"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="
		http://www.springframework.org/schema/beans
		http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/mvc
		http://www.springframework.org/schema/mvc/spring-mvc.xsd">

	<mvc:annotation-driven validator="globalValidator"/>

</beans>
```

To combine global with local validation, simply add one or more local validator\(s\):

```java
@Controller
public class MyController {

	@InitBinder
	protected void initBinder(WebDataBinder binder) {
		binder.addValidators(new FooValidator());
	}

}
```

With this minimal configuration any time an`@Valid`or`@Validated`method argument is encountered, it will be validated by the configured validators. Any validation violations will automatically be exposed as errors in the`BindingResult`accessible as a method argument and also renderable in Spring MVC HTML views.

  


