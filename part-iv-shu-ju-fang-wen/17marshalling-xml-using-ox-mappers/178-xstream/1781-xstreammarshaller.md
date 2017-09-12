### 17.8.1XStreamMarshaller

The`XStreamMarshaller`does not require any configuration, and can be configured in an application context directly. To further customize the XML, you can set an_alias map_, which consists of string aliases mapped to classes:

```java
<beans>
	<bean id="xstreamMarshaller" class="org.springframework.oxm.xstream.XStreamMarshaller">
		<property name="aliases">
			<props>
				<prop key="Flight">org.springframework.oxm.xstream.Flight</prop>
			</props>
		</property>
	</bean>
	...
</beans>
```

| ![](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/warning.png) |
| :--- |
| By default, XStream allows for arbitrary classes to be unmarshalled, which can result in security vulnerabilities. As such, it is_not recommended to use theXStreamMarshallerto unmarshal XML from external sources_\(i.e. the Web\), as this can result in_security vulnerabilities_. If you do use the`XStreamMarshaller`to unmarshal XML from an external source, set the`supportedClasses`property on the`XStreamMarshaller`, like so:`...`This will make sure that only the registered classes are eligible for unmarshalling.Additionally, you can register[custom converters](http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/oxm/xstream/XStreamMarshaller.html#setConverters%28com.thoughtworks.xstream.converters.ConverterMatcher%E2%80%A6%E2%80%8B%29)to make sure that only your supported classes can be unmarshalled. You might want to add a`CatchAllConverter`as the last converter in the list, in addition to converters that explicitly support the domain classes that should be supported. As a result, default XStream converters with lower priorities and possible security vulnerabilities do not get invoked. |

| ![](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png) |
| :--- |
| Note that XStream is an XML serialization library, not a data binding library. Therefore, it has limited namespace support. As such, it is rather unsuitable for usage within Web services. |



