## 18.15Code-based Servlet container initialization

In a Servlet 3.0+ environment, you have the option of configuring the Servlet container programmatically as an alternative or in combination with a`web.xml`file. Below is an example of registering a`DispatcherServlet`:

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

`WebApplicationInitializer`is an interface provided by Spring MVC that ensures your implementation is detected and automatically used to initialize any Servlet 3 container. An abstract base class implementation of`WebApplicationInitializer`named`AbstractDispatcherServletInitializer`makes it even easier to register the`DispatcherServlet`by simply overriding methods to specify the servlet mapping and the location of the`DispatcherServlet`configuration.

This is recommended for applications that use Java-based Spring configuration:

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

If using XML-based Spring configuration, you should extend directly from`AbstractDispatcherServletInitializer`:

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

`AbstractDispatcherServletInitializer`also provides a convenient way to add`Filter`instances and have them automatically mapped to the`DispatcherServlet`:

```java
public class MyWebAppInitializer extends AbstractDispatcherServletInitializer {

	// ...

	@Override
	protected Filter[] getServletFilters() {
		return new Filter[] { new HiddenHttpMethodFilter(), new CharacterEncodingFilter() };
	}

}
```

Each filter is added with a default name based on its concrete type and automatically mapped to the`DispatcherServlet`.

The`isAsyncSupported`protected method of`AbstractDispatcherServletInitializer`provides a single place to enable async support on the`DispatcherServlet`and all filters mapped to it. By default this flag is set to`true`.

Finally, if you need to further customize the`DispatcherServlet`itself, you can override the`createDispatcherServlet`method.

