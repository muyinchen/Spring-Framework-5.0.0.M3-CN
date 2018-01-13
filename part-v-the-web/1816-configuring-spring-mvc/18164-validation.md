### 18.16.4验证

Spring提供了一个[验证器接口](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/validation.html#validator)，可用于在应用程序的所有层进行验证。 在Spring MVC中，您可以将其配置为用作全局`Validator`实例，以便在遇到`@Valid`或`@Validated`控制器方法参数时使用该实例，并且/或者通过`@InitBinder`方法将其用作控制器内的本地`Validator`程序。 全局和本地验证器实例可以组合起来提供复合验证。

Spring还[支持JSR-303 / JSR-349](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/validation.html#validation-beanvalidation-overview)Bean验证，通过`LocalValidatorFactoryBean`使Spring `org.springframework.validation.Validator`接口适应Bean验证`javax.validation.Validator`契约。 这个类可以作为一个全局验证器插入到Spring MVC中，如下所述。

默认情况下，使用`@EnableWebMvc`或者`<mvc:annotation-driven>`当在类路径上检测到一个Bean验证提供程序（如Hibernate Validator）时，通过`LocalValidatorFactoryBean`在Spring MVC中自动注册Bean验证支持。

| ![](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png) |
| :--- |
| 有时将`LocalValidatorFactoryBean`注入到控制器或其他类中很方便。 最简单的方法是声明自己的`@Bean`，并用`@Primary`标记以避免与MVC Java配置提供的冲突。如果您更喜欢使用MVC Java配置中的那个，那么您 将需要重写`WebMvcConfigurationSupport`中的`mvcValidator`方法，并声明该方法显式返回`LocalValidatorFactory`而不是`Validator`。 有关如何切换以扩展提供的配置的信息，请参见[Section18.16.13, “Advanced Customizations with MVC Java Config”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-config-advanced-java)。 |

或者，您可以配置您自己的全局`Validator`实例：

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

和XML：

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

要将全局和本地验证相结合，只需添加一个或多个本地验证程序即可：

```java
@Controller
public class MyController {

    @InitBinder
    protected void initBinder(WebDataBinder binder) {
        binder.addValidators(new FooValidator());
    }

}
```

有了这个最小的配置，任何时候遇到`@Valid`或`@Validated`方法参数，它都将被配置的验证器验证。 任何验证违规将自动作为方法参数可访问的`BindingResult`中的错误公开，也可以在Spring MVC HTML视图中呈现。

