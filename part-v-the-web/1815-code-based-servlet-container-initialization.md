## 18.15基于代码的Servlet容器初始化

在Servlet 3.0+环境中，您可以选择以编程方式配置Servlet容器作为替代方法或与`web.xml`文件结合使用。 以下是一个注册`DispatcherServlet`的例子：

```java
import org.springframework.web.WebApplicationInitializer;

public class MyWebApplicationInitializer implements WebApplicationInitializer {

    @Override
    public void onStartup(ServletContext container) {
        XmlWebApplicationContext appContext = new XmlWebApplicationContext();
        appContext.setConfigLocation("/WEB-INF/spring/dispatcher-config.xml");

        ServletRegistration.Dynamic registration = container.addServlet("dispatcher", new DispatcherServlet(appContext));
        registration.setLoadOnStartup(1);
        registration.addMapping("/");
    }

}
```

`WebApplicationInitializer`是Spring MVC提供的一个接口，它确保您的实现被检测到并自动用来初始化任何Servlet 3容器。 一个名为`AbstractDispatcherServletInitializer`的`WebApplicationInitializer`的抽象基类实现，通过简单地重写方法来指定Servlet映射和`DispatcherServlet`配置的位置，使注册`DispatcherServlet`变得更加容易。

推荐使用基于Java的Spring配置的应用程序：

```java
public class MyWebAppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

    @Override
    protected Class<?>[] getRootConfigClasses() {
        return null;
    }

    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class[] { MyWebConfig.class };
    }

    @Override
    protected String[] getServletMappings() {
        return new String[] { "/" };
    }

}
```

如果使用基于XML的Spring配置，则应该直接从`AbstractDispatcherServletInitializer`扩展：

```java
public class MyWebAppInitializer extends AbstractDispatcherServletInitializer {

    @Override
    protected WebApplicationContext createRootApplicationContext() {
        return null;
    }

    @Override
    protected WebApplicationContext createServletApplicationContext() {
        XmlWebApplicationContext cxt = new XmlWebApplicationContext();
        cxt.setConfigLocation("/WEB-INF/spring/dispatcher-config.xml");
        return cxt;
    }

    @Override
    protected String[] getServletMappings() {
        return new String[] { "/" };
    }

}
```

`AbstractDispatcherServletInitializer`还提供了添加`Filter`实例并将其自动映射到`DispatcherServlet`的便捷方式：

```java
public class MyWebAppInitializer extends AbstractDispatcherServletInitializer {

    // ...

    @Override
    protected Filter[] getServletFilters() {
        return new Filter[] { new HiddenHttpMethodFilter(), new CharacterEncodingFilter() };
    }

}
```

每个过滤器都根据其具体类型添加一个默认名称，并自动映射到`DispatcherServlet`。

`AbstractDispatcherServletInitializer`的`isAsyncSupported`受保护方法提供了一个单独的位置来启用`DispatcherServlet`上的异步支持以及映射到它的所有过滤器。 默认情况下，这个标志被设置为`true`。

最后，如果您需要进一步自定义`DispatcherServlet`本身，则可以覆盖`createDispatcherServlet`方法。

