### 18.16.10回到“Default”Servlet服务资源

这允许将`DispatcherServlet`映射到"/"（从而覆盖容器默认Servlet的映射），同时允许静态资源请求由容器的默认Servlet处理。 它使用URL映射“/\*\*”配置`DefaultServletHttpRequestHandler`，并且相对于其他URL映射的优先级最低。

这个处理程序将把所有请求转发给默认的Servlet。 因此重要的是它保持最后的所有其他URL `HandlerMappings`的顺序。 如果您使用`<mvc:annotation-driven>`，或者如果您正在设置自己的自定义`HandlerMapping`实例，请确保将其`order`属性设置为比`DefaultServletHttpRequestHandler`（即`Integer.MAX_VALUE`）更低的值。

要使用默认设置启用该功能，请使用：

```java
@Configuration
@EnableWebMvc
public class WebConfig extends WebMvcConfigurerAdapter {

    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }

}
```

或者在XML中：

```java
<mvc:default-servlet-handler/>
```

覆盖“/” Servlet映射的警告是，默认Servlet的`RequestDispatcher`必须通过名称而不是路径来检索。 `DefaultServletHttpRequestHandler`将尝试在启动时使用大多数主要Servlet容器（包括Tomcat，Jetty，GlassFish，JBoss，Resin，WebLogic和WebSphere）的已知名称列表来自动检测容器的默认Servlet。 如果默认的Servlet已经被自定义配置了一个不同的名字，或者当默认的Servlet名字是未知的时候使用了一个不同的Servlet容器，那么默认的Servlet的名字必须被显式地提供，如下例所示：

```java
@Configuration
@EnableWebMvc
public class WebConfig extends WebMvcConfigurerAdapter {

    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable("myCustomDefaultServlet");
    }

}
```

或者在XML中：

```java
<mvc:default-servlet-handler default-servlet-name="myCustomDefaultServlet"/>
```



