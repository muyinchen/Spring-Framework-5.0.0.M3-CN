### 19.4.2上下文配置

通过将相关的configurer bean定义添加到您的`'*-servlet.xml'`来初始化一个合适的配置，如下所示：

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
| 对于非web应用程序，将`FreeMarkerConfigurationFactoryBean`添加到您的应用程序上下文定义文件中。 |



