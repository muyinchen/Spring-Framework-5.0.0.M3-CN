### 18.16.7视图控制器

这是定义一个`ParameterizableViewController`的快捷方式，可以在调用时立即转发到视图。 在静态情况下使用它，在视图生成响应之前没有执行Java控制器逻辑。

在Java中将`"/"`的请求转发到名为`"home"`的视图的示例：

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

XML中的相同内容使用`<mvc:view-controller>`元素：

```java
<mvc:view-controller path="/" view-name="home"/>
```



