### 18.16.14使用MVC命名空间进行高级自定义

对您创建的配置进行细粒度的控制对于MVC命名空间来说有点难度。

如果您需要这样做，而不是复制它提供的配置，请考虑配置一个`BeanPostProcessor`，该`BeanPostProcessor`检测要按类型自定义的bean，然后根据需要修改其属性。 例如：

```java
@Component
public class MyPostProcessor implements BeanPostProcessor {

    public Object postProcessBeforeInitialization(Object bean, String name) throws BeansException {
        if (bean instanceof RequestMappingHandlerAdapter) {
            // Modify properties of the adapter
        }
    }

}
```

请注意，`MyPostProcessor`需要包含在`<component scan />`中才能被检测到，或者如果您愿意，可以使用XML bean声明显式声明它。

