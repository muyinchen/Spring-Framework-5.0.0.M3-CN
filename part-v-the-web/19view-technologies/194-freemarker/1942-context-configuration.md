### 19.4.2Context configuration

A suitable configuration is initialized by adding the relevant configurer bean definition to your`'*-servlet.xml'`as shown below:

```java
<!-- freemarker config -->
<bean id="freemarkerConfig" class="org.springframework.web.servlet.view.freemarker.FreeMarkerConfigurer">
	<property name="templateLoaderPath" value="/WEB-INF/freemarker/"/>
</bean>

<!--
View resolvers can also be configured with ResourceBundles or XML files. If you need
different view resolving based on Locale, you have to use the resource bundle resolver.
-->
<bean id="viewResolver" class="org.springframework.web.servlet.view.freemarker.FreeMarkerViewResolver">
	<property name="cache" value="true"/>
	<property name="prefix" value=""/>
	<property name="suffix" value=".ftl"/>
</bean>
```

| ![](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png) |
| :--- |
| For non web-apps add a`FreeMarkerConfigurationFactoryBean`to your application context definition file. |



