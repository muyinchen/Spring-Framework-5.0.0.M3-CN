### 18.16.2定制提供的配置

要在Java中定制默认配置，只需实现`WebMvcConfigurer`接口，或者更可能扩展`WebMvcConfigurerAdapter`类并覆盖所需的方法：

```java
@Configuration
@EnableWebMvc
public class WebConfig extends WebMvcConfigurerAdapter {

    // Override configuration methods...

}
```

要自定义`<mvc:annotation-driven/>`的缺省配置，请检查它支持的属性和子元素。 您可以查看[Spring MVC XML模式](http://schema.spring.io/mvc/spring-mvc.xsd)或使用IDE的代码完成功能来发现可用的属性和子元素。

