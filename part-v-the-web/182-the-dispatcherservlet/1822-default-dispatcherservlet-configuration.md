### 18.2.2 Default DispatcherServlet Configuration

As mentioned in the previous section for each special bean the`DispatcherServlet`maintains a list of implementations to use by default. This information is kept in the file`DispatcherServlet.properties`in the package`org.springframework.web.servlet`.

All special beans have some reasonable defaults of their own. Sooner or later though you’ll need to customize one or more of the properties these beans provide. For example it’s quite common to configure an`InternalResourceViewResolver`settings its`prefix`property to the parent location of view files.

Regardless of the details, the important concept to understand here is that once you configure a special bean such as an`InternalResourceViewResolver`in your`WebApplicationContext`, you effectively override the list of default implementations that would have been used otherwise for that special bean type. For example if you configure an`InternalResourceViewResolver`, the default list of`ViewResolver`implementations is ignored.

In[Section 18.16, “Configuring Spring MVC”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-config)you’ll learn about other options for configuring Spring MVC including MVC Java config and the MVC XML namespace both of which provide a simple starting point and assume little knowledge of how Spring MVC works. Regardless of how you choose to configure your application, the concepts explained in this section are fundamental and should be of help to you.

