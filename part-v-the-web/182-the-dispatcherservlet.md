## 18.2The DispatcherServlet

Spring’s web MVC framework is, like many other web MVC frameworks, request-driven, designed around a central Servlet that dispatches requests to controllers and offers other functionality that facilitates the development of web applications. Spring’s`DispatcherServlet`however, does more than just that. It is completely integrated with the Spring IoC container and as such allows you to use every other feature that Spring has.

The request processing workflow of the Spring Web MVC`DispatcherServlet`is illustrated in the following diagram. The pattern-savvy reader will recognize that the`DispatcherServlet`is an expression of the "Front Controller" design pattern \(this is a pattern that Spring Web MVC shares with many other leading web frameworks\).

**Figure18.1.The request processing workflow in Spring Web MVC \(high level\)**

![](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/mvc.png)

The`DispatcherServlet`is an actual`Servlet`\(it inherits from the`HttpServlet`base class\), and as such is declared in your web application. You need to map requests that you want the`DispatcherServlet`to handle, by using a URL mapping. Here is a standard Java EE Servlet configuration in a Servlet 3.0+ environment:

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

In the preceding example, all requests starting with`/example`will be handled by the`DispatcherServlet`instance named`example`.

`WebApplicationInitializer`is an interface provided by Spring MVC that ensures your code-based configuration is detected and automatically used to initialize any Servlet 3 container. An abstract base class implementation of this interface named`AbstractAnnotationConfigDispatcherServletInitializer`makes it even easier to register the`DispatcherServlet`by simply specifying its servlet mapping and listing configuration classes - it’s even the recommended way to set up your Spring MVC application. See[Code-based Servlet container initialization](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-container-config)for more details.

The`DispatcherServlet`is an actual`Servlet`\(it inherits from the`HttpServlet`base class\), and as such is declared in the`web.xml`of your web application. You need to map requests that you want the`DispatcherServlet`to handle, by using a URL mapping in the same`web.xml`file. This is standard Java EE Servlet configuration; the following example shows such a`DispatcherServlet`declaration and mapping:

Below is the`web.xml`equivalent of the above code based example:

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

As detailed in[Section3.15, “Additional Capabilities of the ApplicationContext”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/beans.html#context-introduction),`ApplicationContext`instances in Spring can be scoped. In the Web MVC framework, each`DispatcherServlet`has its own`WebApplicationContext`, which inherits all the beans already defined in the root`WebApplicationContext`. The root`WebApplicationContext`should contain all the infrastructure beans that should be shared between your other contexts and Servlet instances. These inherited beans can be overridden in the servlet-specific scope, and you can define new scope-specific beans local to a given Servlet instance.

**Figure18.2.Typical context hierarchy in Spring Web MVC**

![](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/mvc-context-hierarchy.png)

Upon initialization of a`DispatcherServlet`, Spring MVC looks for a file named_\[servlet-name\]-servlet.xml_in the`WEB-INF`directory of your web application and creates the beans defined there, overriding the definitions of any beans defined with the same name in the global scope.

Consider the following`DispatcherServlet`Servlet configuration \(in the`web.xml`file\):

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

With the above Servlet configuration in place, you will need to have a file called`/WEB-INF/golfing-servlet.xml`in your application; this file will contain all of your Spring Web MVC-specific components \(beans\). You can change the exact location of this configuration file through a Servlet initialization parameter \(see below for details\).

It is also possible to have just one root context for single DispatcherServlet scenarios.

**Figure18.3.Single root context in Spring Web MVC**

![](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/mvc-root-context.png)

This can be configured by setting an empty contextConfigLocation servlet init parameter, as shown below:

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

The`WebApplicationContext`is an extension of the plain`ApplicationContext`that has some extra features necessary for web applications. It differs from a normal`ApplicationContext`in that it is capable of resolving themes \(see[Section18.9, “Using themes”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-themeresolver)\), and that it knows which Servlet it is associated with \(by having a link to the`ServletContext`\). The`WebApplicationContext`is bound in the`ServletContext`, and by using static methods on the`RequestContextUtils`class you can always look up the`WebApplicationContext`if you need access to it.

Note that we can achieve the same with java-based configurations:

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

  


