### 19.3.1配置

配置 Groovy Markup Template Engine相当容易:

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

使用MVC命名空间的XML文本:

```java
<mvc:annotation-driven/>

<mvc:view-resolvers>
    <mvc:groovy/>
</mvc:view-resolvers>

<mvc:groovy-configurer resource-loader-path="/WEB-INF/"/>
```



