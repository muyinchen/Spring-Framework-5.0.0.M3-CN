## 18.16 Configuring Spring MVC

[Section 18.2.1, “Special Bean Types In the WebApplicationContext”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-servlet-special-bean-types)and[Section 18.2.2, “Default DispatcherServlet Configuration”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-servlet-config)explained about Spring MVC’s special beans and the default implementations used by the`DispatcherServlet`. In this section you’ll learn about two additional ways of configuring Spring MVC. Namely the MVC Java config and the MVC XML namespace.

The MVC Java config and the MVC namespace provide similar default configuration that overrides the`DispatcherServlet`defaults. The goal is to spare most applications from having to create the same configuration and also to provide higher-level constructs for configuring Spring MVC that serve as a simple starting point and require little or no prior knowledge of the underlying configuration.

You can choose either the MVC Java config or the MVC namespace depending on your preference. Also as you will see further below, with the MVC Java config it is easier to see the underlying configuration as well as to make fine-grained customizations directly to the created Spring MVC beans. But let’s start from the beginning.

