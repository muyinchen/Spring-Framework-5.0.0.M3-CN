### 18.16.3Conversion and Formatting

By default formatters for`Number`and`Date`types are installed, including support for the`@NumberFormat`and`@DateTimeFormat`annotations. Full support for the Joda Time formatting library is also installed if Joda Time is present on the classpath. To register custom formatters and converters, override the`addFormatters`method:

```java
@Configuration
@EnableWebMvc
public class WebConfig extends WebMvcConfigurerAdapter {

	@Override
	public void addFormatters(FormatterRegistry registry) {
		// Add formatters and/or converters
	}

}
```

In the MVC namespace the same defaults apply when```is added. To register custom formatters and converters simply supply a``ConversionService\`:

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:mvc="http://www.springframework.org/schema/mvc"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="
		http://www.springframework.org/schema/beans
		http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/mvc
		http://www.springframework.org/schema/mvc/spring-mvc.xsd">

	<mvc:annotation-driven conversion-service="conversionService"/>

	<bean id="conversionService"
			class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
		<property name="converters">
			<set>
				<bean class="org.example.MyConverter"/>
			</set>
		</property>
		<property name="formatters">
			<set>
				<bean class="org.example.MyFormatter"/>
				<bean class="org.example.MyAnnotationFormatterFactory"/>
			</set>
		</property>
		<property name="formatterRegistrars">
			<set>
				<bean class="org.example.MyFormatterRegistrar"/>
			</set>
		</property>
	</bean>

</beans>
```

| ![](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png) |
| :--- |
| See[Section5.6.4, “FormatterRegistrar SPI”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/validation.html#format-FormatterRegistrar-SPI)and the`FormattingConversionServiceFactoryBean`for more information on when to use FormatterRegistrars. |



