### 18.10.2Using a MultipartResolver with_Commons FileUpload_

The following example shows how to use the`CommonsMultipartResolver`:

```java
<bean id="multipartResolver"
		class="org.springframework.web.multipart.commons.CommonsMultipartResolver">

	<!-- one of the properties available; the maximum file size in bytes -->
	<property name="maxUploadSize" value="100000"/>

</bean>
```

Of course you also need to put the appropriate jars in your classpath for the multipart resolver to work. In the case of the`CommonsMultipartResolver`, you need to use`commons-fileupload.jar`.

When the Spring`DispatcherServlet`detects a multi-part request, it activates the resolver that has been declared in your context and hands over the request. The resolver then wraps the current`HttpServletRequest`into a`MultipartHttpServletRequest`that supports multipart file uploads. Using the`MultipartHttpServletRequest`, you can get information about the multiparts contained by this request and actually get access to the multipart files themselves in your controllers.

