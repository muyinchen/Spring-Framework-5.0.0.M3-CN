### 18.10.2与Commons FileUpload一起使用MultipartResolver

以下示例显示如何使用`CommonsMultipartResolver`：

```java
<bean id="multipartResolver"
        class="org.springframework.web.multipart.commons.CommonsMultipartResolver">

    <!-- 可用属性之一; 最大文件大小（以字节为单位） -->
    <property name="maxUploadSize" value="100000"/>

</bean>
```

当然，你也需要在你的类路径中放入适当的jar，以便多部分解析器工作。 在`CommonsMultipartResolver`的情况下，你需要使用`commons-fileupload.jar`。

当Spring `DispatcherServlet`检测到多部分请求时，它会激活已经在上下文中声明的解析器并移交请求。 然后，解析器将当前的`HttpServletRequest`包装成支持多部分文件上传的`MultipartHttpServletRequest`。 使用`MultipartHttpServletRequest`，您可以获得有关此请求包含的多部分的信息，实际上可以在您的控制器中访问多部分文件。

