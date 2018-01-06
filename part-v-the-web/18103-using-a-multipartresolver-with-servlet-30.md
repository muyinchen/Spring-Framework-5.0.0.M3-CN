### 18.10.3在Servlet 3.0中使用MultipartResolver

为了使用基于Servlet 3.0的多部分解析，您需要在`web.xml`中使用`“multipart-config”`部分标记`DispatcherServlet`，或者使用编程Servlet注册中的`javax.servlet.MultipartConfigElement`标记，或者使用自定义的Servlet类 可能在你的Servlet类中有一个`javax.servlet.annotation.MultipartConfig`注解。 配置设置（如最大大小或存储位置）需要在该Servlet注册级别应用，因为Servlet 3.0不允许从`MultipartResolver`完成这些设置。

一旦使用上述方式之一启用了Servlet 3.0 multipart解析，您可以将`StandardServletMultipartResolver`添加到您的Spring配置中：

```java
<bean id="multipartResolver"
        class="org.springframework.web.multipart.support.StandardServletMultipartResolver">
</bean>
```



