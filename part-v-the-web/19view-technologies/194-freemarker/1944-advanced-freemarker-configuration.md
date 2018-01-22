### 19.4.4高级FreeMarker配置

通过在`FreeMarkerConfigurer` bean上设置合适的bean属性，FreeMarker的'Settings'和'SharedVariables'可以直接传递给Spring管理的FreeMarker`Configuration`对象。 `freemarkerSettings`属性需要一个`java.util.Properties`对象，`freemarkerVariables`属性需要一个`java.util.Map`。

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

有关适用于`Configuration`对象的设置和变量的详细信息，请参阅FreeMarker文档。

