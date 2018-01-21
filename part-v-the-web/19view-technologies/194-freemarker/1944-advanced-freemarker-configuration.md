### 19.4.4Advanced FreeMarker configuration

FreeMarker 'Settings' and 'SharedVariables' can be passed directly to the FreeMarker`Configuration`object managed by Spring by setting the appropriate bean properties on the`FreeMarkerConfigurer`bean. The`freemarkerSettings`property requires a`java.util.Properties`object and the`freemarkerVariables`property requires a`java.util.Map`.

```java
<bean id="freemarkerConfig" class="org.springframework.web.servlet.view.freemarker.FreeMarkerConfigurer">
	<property name="templateLoaderPath" value="/WEB-INF/freemarker/"/>
	<property name="freemarkerVariables">
		<map>
			<entry key="xml_escape" value-ref="fmXmlEscape"/>
		</map>
	</property>
</bean>

<bean id="fmXmlEscape" class="freemarker.template.utility.XmlEscape"/>
```

See the FreeMarker documentation for details of settings and variables as they apply to the`Configuration`object.

  


