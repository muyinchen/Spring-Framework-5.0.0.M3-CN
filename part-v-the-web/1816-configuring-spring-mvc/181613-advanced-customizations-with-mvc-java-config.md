### 18.16.13使用MVC Java配置进行高级自定义

从上面的例子可以看出，MVC Java配置和MVC命名空间提供了更高层次的构造，不需要深入了解为您创建的底层bean。 相反，它可以帮助您专注于您的应用程序需求。 但是，在某些时候，您可能需要更细粒度的控制，或者您可能只是想了解底层配置。

更细粒度控制的第一步是查看为您创建的底层bean。 在MVC Java配置中，您可以在`WebMvcConfigurationSupport`中看到javadocs和`@Bean`方法。 此类中的配置通过`@EnableWebMvc`注解自动导入。 实际上，如果您打开`@EnableWebMvc`，则可以看到`@Import`语句。

更细粒度控制的下一步是定制`WebMvcConfigurationSupport`中创建的一个bean的属性，或者提供自己的实例。 这需要两件事 - 删除`@EnableWebMvc`注解，以防止导入，然后从`WebMvcConfigurationSupport`的子类`DelegatingWebMvcConfiguration`扩展。 这里是一个例子：

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
| 一个应用程序应该只有一个配置来扩展`DelegatingWebMvcConfiguration`或一个`@EnableWebMvc`注解类，因为它们都注册了相同的底层bean。用这种方式修改bean并不妨碍你使用本节前面所示的任何更高级的结构。`WebMvcConfigurerAdapter`子类和`WebMvcConfigurer`实现仍在使用中。 |



