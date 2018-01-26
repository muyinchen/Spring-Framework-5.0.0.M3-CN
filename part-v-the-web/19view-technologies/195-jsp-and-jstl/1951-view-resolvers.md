### 19.5.1视图解析

就像其他任何与Spring集成的视图技术一样，对于JSP，您需要一个视图解析器来解析视图。 在使用JSP进行开发时，最常用的视图解析器是`InternalResourceViewResolver`和`ResourceBundleViewResolver`。 两者都在`WebApplicationContext`中声明：

```java
<!-- the ResourceBundleViewResolver -->
<bean id="viewResolver" class="org.springframework.web.servlet.view.ResourceBundleViewResolver">
    <property name="basename" value="views"/>
</bean>

# And a sample properties file is uses (views.properties in WEB-INF/classes):
welcome.(class)=org.springframework.web.servlet.view.JstlView
welcome.url=/WEB-INF/jsp/welcome.jsp

productList.(class)=org.springframework.web.servlet.view.JstlView
productList.url=/WEB-INF/jsp/productlist.jsp
```

正如你所看到的，`ResourceBundleViewResolver`需要一个属性文件来定义映射到1）类和2）URL的视图名称。 使用`ResourceBundleViewResolver`，您可以只使用一个解析器混合不同类型的视图。

```java
<bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="viewClass" value="org.springframework.web.servlet.view.JstlView"/>
    <property name="prefix" value="/WEB-INF/jsp/"/>
    <property name="suffix" value=".jsp"/>
</bean>
```

如上所述，可以将`InternalResourceBundleViewResolver`配置为使用JSP。 作为最佳做法，我们强烈建议将您的JSP文件放置在`'WEB-INF'`目录下的目录中，这样客户端就不能直接访问了。

