## 18.2DispatcherServlet

Spring的Web MVC框架与许多其他Web MVC框架一样，以请求为驱动，围绕一个中央Servlet设计，将请求发送给控制器，并提供了其他促进Web应用程序开发的功能。然而， Spring 的`DispatcherServlet` 做得更多.它和 Spring IoC 容器整合一起，它允许你使用Spring 每个特性.

Spring Web MVC `DispatcherServlet`的请求处理工作流程如下图所示。 对设计模式熟悉的读者将会认识到，`DispatcherServlet`是“前端控制器”设计模式的表达（这是Spring Web MVC与许多其他领先的Web框架共享的模式）。

**Figure18.1.在Spring Web MVC 中请求处理流程\(high level\)**

![](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/mvc.png)

`DispatcherServlet`是一个实际的`Servlet`（它继承自`HttpServlet`基类），因此在Web应用程序中被声明。 您需要使用URL映射来映射要`DispatcherServlet`处理的请求。 以下是Servlet 3.0+环境中的标准Java EE Servlet配置：

```java
public class MyWebApplicationInitializer implements WebApplicationInitializer {

    @Override
    public void onStartup(ServletContext container) {
        ServletRegistration.Dynamic registration = container.addServlet("example", new DispatcherServlet());
        registration.setLoadOnStartup(1);
        registration.addMapping("/example/*");
    }

}
```

在前面的示例中，以`/example`开头的所有请求都将由名为`Example`的`DispatcherServlet`实例处理。

`WebApplicationInitializer`是由Spring MVC提供的接口，可确保您的基于代码的配置被检测并自动用于初始化任何Servlet 3容器。这个名为`AbstractAnnotationConfigDispatcherServletInitializer`的接口的抽象基类实现通过简单地指定其servlet映射和列出配置类来更容易地注册`DispatcherServlet`，甚至建议您设置Spring MVC应用程序。有关更多详细信息，请参阅[基于代码的Servlet容器初始化](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-container-config)。

`DispatcherServlet`是一个实际的Servlet（它继承自HttpServlet基类），因此在Web应用程序的`web.xml`中声明。您需要通过使用同一`web.xml`文件中的URL映射来映射要`DispatcherServlet`处理的请求。这是标准的Java EE Servlet配置。

以下示例`web.xml`显示了这样的`DispatcherServlet`声明和映射：

```java
<web-app>
    <servlet>
        <servlet-name>example</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>example</servlet-name>
        <url-pattern>/example/*</url-pattern>
    </servlet-mapping>

</web-app>
```

如[第3.15节“ApplicationContext的附加功能”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/beans.html#context-introduction)中所述，Spring中的`ApplicationContext`实例可以被限定。 在Web MVC框架中，每个`DispatcherServlet`都有自己的`WebApplicationContext`，它继承了已经在根`WebApplicationContext`中定义的所有bean。 根`WebApplicationContext`应该包含应该在其他上下文和Servlet实例之间共享的所有基础架构bean。 这些继承的bean可以在特定于servlet的范围内被覆盖，您可以在给定的Servlet实例本地定义新的范围特定的bean。

**Figure18.2.Spring Web MVC中的典型上下文层次结构**

![](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/mvc-context-hierarchy.png)

在初始化`DispatcherServlet`时，Spring MVC将在Web应用程序的`WEB-INF`目录中查找名为\[servlet-name\] -servlet.xml的文件，并创建在那里定义的bean，覆盖使用相同名称定义的任何bean的定义 在全球范围内。

请考虑以下`DispatcherServlet` Servlet配置（在`web.xml`文件中）：

```java
<web-app>
    <servlet>
        <servlet-name>golfing</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>golfing</servlet-name>
        <url-pattern>/golfing/*</url-pattern>
    </servlet-mapping>
</web-app>
```

使用上述Servlet配置，您将需要在应用程序中有一个名为`/WEB-INF/golfing-servlet.xml`的文件; 该文件将包含您所有的Spring Web MVC特定组件（bean）。 您可以通过Servlet初始化参数更改此配置文件的确切位置（有关详细信息，请参阅下文）。

单个DispatcherServlet方案也可能只有一个根上下文。

**Figure18.3.Single root context in Spring Web MVC**

![](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/mvc-root-context.png)

这可以通过设置一个空的ContextConfigLocation servlet init参数进行配置，如下所示：

```java
<web-app>
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/root-context.xml</param-value>
    </context-param>
    <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value></param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <url-pattern>/*</url-pattern>
    </servlet-mapping>
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
</web-app>
```

`WebApplicationContext`是普通`ApplicationContext`的扩展，它具有Web应用程序所需的一些额外功能。 它与正常的`ApplicationContext`不同之处在于它能够解析主题（参见[第18.9节“使用主题”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-themeresolver)），并且知道它与哪个Servlet相关联（通过连接到`ServletContext`）。 `WebApplicationContext`绑定在`ServletContext`中，并且通过在`RequestContextUtils`类上使用静态方法，您可以随时查找`WebApplicationContext`，如果您需要访问它。

请注意，我们可以通过基于Java的配置实现相同的方式：

```java
public class GolfingWebAppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

    @Override
    protected Class<?>[] getRootConfigClasses() {
        // GolfingAppConfig defines beans that would be in root-context.xml
        return new Class[] { GolfingAppConfig.class };
    }

    @Override
    protected Class<?>[] getServletConfigClasses() {
        // GolfingWebConfig defines beans that would be in golfing-servlet.xml
        return new Class[] { GolfingWebConfig.class };
    }

    @Override
    protected String[] getServletMappings() {
        return new String[] { "/golfing/*" };
    }

}
```



