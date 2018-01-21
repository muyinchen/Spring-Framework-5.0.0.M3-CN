### 19.5.1View resolvers

Just as with any other view technology you’re integrating with Spring, for JSPs you’ll need a view resolver that will resolve your views. The most commonly used view resolvers when developing with JSPs are the`InternalResourceViewResolver`and the`ResourceBundleViewResolver`. Both are declared in the`WebApplicationContext`:

```java
<!-- the ResourceBundleViewResolver -->
<bean id="viewResolver" class="org.springframework.web.servlet.view.ResourceBundleViewResolver">
	<property name="basename" value="views"/>
</bean>

# And a sample properties file is uses (views.properties in WEB-INF/classes):
welcome.(class)=org.springframework.web.servlet.view.JstlView
welcome.url=/WEB-INF/jsp/welcome.jsp

productList.(class)=org.springframework.web.servlet.view.JstlView
productList.url=/WEB-INF/jsp/productlist.jsp
```

As you can see, the`ResourceBundleViewResolver`needs a properties file defining the view names mapped to 1\) a class and 2\) a URL. With a`ResourceBundleViewResolver`you can mix different types of views using only one resolver.

```java
<bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
	<property name="viewClass" value="org.springframework.web.servlet.view.JstlView"/>
	<property name="prefix" value="/WEB-INF/jsp/"/>
	<property name="suffix" value=".jsp"/>
</bean>
```

The`InternalResourceBundleViewResolver`can be configured for using JSPs as described above. As a best practice, we strongly encourage placing your JSP files in a directory under the`'WEB-INF'`directory, so there can be no direct access by clients.

