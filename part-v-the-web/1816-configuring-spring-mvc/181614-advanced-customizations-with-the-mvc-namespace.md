### 18.16.14Advanced Customizations with the MVC Namespace

Fine-grained control over the configuration created for you is a bit harder with the MVC namespace.

If you do need to do that, rather than replicating the configuration it provides, consider configuring a`BeanPostProcessor`that detects the bean you want to customize by type and then modifying its properties as necessary. For example:

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

Note that`MyPostProcessor`needs to be included in an `<component scan/>` in order for it to be detected or if you prefer you can declare it explicitly with an XML bean declaration.

  


