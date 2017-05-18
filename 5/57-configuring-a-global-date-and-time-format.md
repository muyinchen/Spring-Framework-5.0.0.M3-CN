## 5.7 Configuring a global date & time format

By default, date and time fields that are not annotated with `@DateTimeFormat` are converted from strings using the `DateFormat.SHORT` style. If you prefer, you can change this by defining your own global format.

You will need to ensure that Spring does not register default formatters, and instead you should register all formatters manually. Use the`org.springframework.format.datetime.joda.JodaTimeFormatterRegistrar` or `org.springframework.format.datetime.DateFormatterRegistrar` class depending on whether you use the Joda Time library.

For example, the following Java configuration will register a global ' `yyyyMMdd’ format. This example does not depend on the Joda Time library:

```java
@Configuration
public class AppConfig {

	@Bean
	public FormattingConversionService conversionService() {

		// Use the DefaultFormattingConversionService but do not register defaults
		DefaultFormattingConversionService conversionService = new DefaultFormattingConversionService(false);

		// Ensure @NumberFormat is still supported
		conversionService.addFormatterForFieldAnnotation(new NumberFormatAnnotationFormatterFactory());

		// Register date conversion with a specific global format
		DateFormatterRegistrar registrar = new DateFormatterRegistrar();
		registrar.setFormatter(new DateFormatter("yyyyMMdd"));
		registrar.registerFormatters(conversionService);

		return conversionService;
	}
}
```

If you prefer XML based configuration you can use a `FormattingConversionServiceFactoryBean`. Here is the same example, this time using Joda Time:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="
		http://www.springframework.org/schema/beans
		http://www.springframework.org/schema/beans/spring-beans.xsd>

	<bean id="conversionService" class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
		<property name="registerDefaultFormatters" value="false" />
		<property name="formatters">
			<set>
				<bean class="org.springframework.format.number.NumberFormatAnnotationFormatterFactory" />
			</set>
		</property>
		<property name="formatterRegistrars">
			<set>
				<bean class="org.springframework.format.datetime.joda.JodaTimeFormatterRegistrar">
					<property name="dateFormatter">
						<bean class="org.springframework.format.datetime.joda.DateTimeFormatterFactoryBean">
							<property name="pattern" value="yyyyMMdd"/>
						</bean>
					</property>
				</bean>
			</set>
		</property>
	</bean>
</beans>
```

| ![[Note]](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| ---------------------------------------- |
| Joda Time provides separate distinct types to represent `date`, `time` and `date-time` values. The `dateFormatter`, `timeFormatter` and `dateTimeFormatter` properties of the `JodaTimeFormatterRegistrar` should be used to configure the different formats for each type. The `DateTimeFormatterFactoryBean` provides a convenient way to create formatters. |

If you are using Spring MVC remember to explicitly configure the conversion service that is used. For Java based `@Configuration` this means extending the`WebMvcConfigurationSupport` class and overriding the `mvcConversionService()` method. For XML you should use the `'conversion-service'` attribute of the`mvc:annotation-driven` element. See [Section 18.16.3, “Conversion and Formatting”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#mvc-config-conversion) for details.