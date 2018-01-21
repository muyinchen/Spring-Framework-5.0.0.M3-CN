### 19.8.2How to integrate Tiles

To be able to use Tiles, you have to configure it using files containing definitions \(for basic information on definitions and other Tiles concepts, please have a look at[http://tiles.apache.org](https://tiles.apache.org/)\). In Spring this is done using the`TilesConfigurer`. Have a look at the following piece of example ApplicationContext configuration:

```java
<bean id="tilesConfigurer" class="org.springframework.web.servlet.view.tiles3.TilesConfigurer">
	<property name="definitions">
		<list>
			<value>/WEB-INF/defs/general.xml</value>
			<value>/WEB-INF/defs/widgets.xml</value>
			<value>/WEB-INF/defs/administrator.xml</value>
			<value>/WEB-INF/defs/customer.xml</value>
			<value>/WEB-INF/defs/templates.xml</value>
		</list>
	</property>
</bean>
```

As you can see, there are five files containing definitions, which are all located in the`'WEB-INF/defs'`directory. At initialization of the`WebApplicationContext`, the files will be loaded and the definitions factory will be initialized. After that has been done, the Tiles includes in the definition files can be used as views within your Spring web application. To be able to use the views you have to have a`ViewResolver`just as with any other view technology used with Spring. Below you can find two possibilities, the`UrlBasedViewResolver`and the`ResourceBundleViewResolver`.

You can specify locale specific Tiles definitions by adding an underscore and then the locale. For example:

```java
<bean id="tilesConfigurer" class="org.springframework.web.servlet.view.tiles3.TilesConfigurer">
	<property name="definitions">
		<list>
			<value>/WEB-INF/defs/tiles.xml</value>
			<value>/WEB-INF/defs/tiles_fr_FR.xml</value>
		</list>
	</property>
</bean>
```

With this configuration,`tiles_fr_FR.xml`will be used for requests with the`fr_FR`locale, and`tiles.xml`will be used by default.

| ![](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png) |
| :--- |
| Since underscores are used to indicate locales, it is recommended to avoid using them otherwise in the file names for Tiles definitions. |

#### UrlBasedViewResolver

The`UrlBasedViewResolver`instantiates the given`viewClass`for each view it has to resolve.

```js
<bean id="viewResolver" class="org.springframework.web.servlet.view.UrlBasedViewResolver">
	<property name="viewClass" value="org.springframework.web.servlet.view.tiles3.TilesView"/>
</bean>
```

#### ResourceBundleViewResolver

The`ResourceBundleViewResolver`has to be provided with a property file containing viewnames and viewclasses the resolver can use:

```java
<bean id="viewResolver" class="org.springframework.web.servlet.view.ResourceBundleViewResolver">
	<property name="basename" value="views"/>
</bean>
```

```js
...
welcomeView.(class)=org.springframework.web.servlet.view.tiles3.TilesView
welcomeView.url=welcome (this is the name of a Tiles definition)

vetsView.(class)=org.springframework.web.servlet.view.tiles3.TilesView
vetsView.url=vetsView (again, this is the name of a Tiles definition)

findOwnersForm.(class)=org.springframework.web.servlet.view.JstlView
findOwnersForm.url=/WEB-INF/jsp/findOwners.jsp
...
```

As you can see, when using the`ResourceBundleViewResolver`, you can easily mix different view technologies.

Note that the`TilesView`class supports JSTL \(the JSP Standard Tag Library\) out of the box.

#### SimpleSpringPreparerFactory and SpringBeanPreparerFactory

As an advanced feature, Spring also supports two special Tiles`PreparerFactory`implementations. Check out the Tiles documentation for details on how to use`ViewPreparer`references in your Tiles definition files.

Specify`SimpleSpringPreparerFactory`to autowire ViewPreparer instances based on specified preparer classes, applying Spring’s container callbacks as well as applying configured Spring BeanPostProcessors. If Spring’s context-wide annotation-config has been activated, annotations in ViewPreparer classes will be automatically detected and applied. Note that this expects preparer_classes_in the Tiles definition files, just like the default`PreparerFactory`does.

Specify`SpringBeanPreparerFactory`to operate on specified preparer_names_instead of classes, obtaining the corresponding Spring bean from the DispatcherServlet’s application context. The full bean creation process will be in the control of the Spring application context in this case, allowing for the use of explicit dependency injection configuration, scoped beans etc. Note that you need to define one Spring bean definition per preparer name \(as used in your Tiles definitions\).

```java
<bean id="tilesConfigurer" class="org.springframework.web.servlet.view.tiles3.TilesConfigurer">
	<property name="definitions">
		<list>
			<value>/WEB-INF/defs/general.xml</value>
			<value>/WEB-INF/defs/widgets.xml</value>
			<value>/WEB-INF/defs/administrator.xml</value>
			<value>/WEB-INF/defs/customer.xml</value>
			<value>/WEB-INF/defs/templates.xml</value>
		</list>
	</property>

	<!-- resolving preparer names as Spring bean definition names -->
	<property name="preparerFactoryClass"
			value="org.springframework.web.servlet.view.tiles3.SpringBeanPreparerFactory"/>

</bean>
```



