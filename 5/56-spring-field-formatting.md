## 5.6 Spring Field Formatting

As discussed in the previous section, [`core.convert`](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#core-convert) is a general-purpose type conversion system. It provides a unified ConversionService API as well as a strongly-typed Converter SPI for implementing conversion logic from one type to another. A Spring Container uses this system to bind bean property values. In addition, both the Spring Expression Language (SpEL) and DataBinder use this system to bind field values. For example, when SpEL needs to coerce a `Short` to a `Long` to complete an `expression.setValue(Object bean, Object value)` attempt, the core.convert system performs the coercion.

Now consider the type conversion requirements of a typical client environment such as a web or desktop application. In such environments, you typically convert *from String* to support the client postback process, as well as back *to String* to support the view rendering process. In addition, you often need to localize String values. The more general *core.convert* Converter SPI does not address such *formatting* requirements directly. To directly address them, Spring 3 introduces a convenient Formatter SPI that provides a simple and robust alternative to PropertyEditors for client environments.

In general, use the Converter SPI when you need to implement general-purpose type conversion logic; for example, for converting between a java.util.Date and and java.lang.Long. Use the Formatter SPI when you’re working in a client environment, such as a web application, and need to parse and print localized field values. The ConversionService provides a unified type conversion API for both SPIs.

### 5.6.1 Formatter SPI

The Formatter SPI to implement field formatting logic is simple and strongly typed:

```java
package org.springframework.format;

public interface Formatter<T> extends Printer<T>, Parser<T> {
}
```

Where Formatter extends from the Printer and Parser building-block interfaces:

```java
public interface Printer<T> {
	String print(T fieldValue, Locale locale);
}
```

```java
import java.text.ParseException;

public interface Parser<T> {
	T parse(String clientValue, Locale locale) throws ParseException;
}
```

To create your own Formatter, simply implement the Formatter interface above. Parameterize T to be the type of object you wish to format, for example,`java.util.Date`. Implement the `print()` operation to print an instance of T for display in the client locale. Implement the `parse()` operation to parse an instance of T from the formatted representation returned from the client locale. Your Formatter should throw a ParseException or IllegalArgumentException if a parse attempt fails. Take care to ensure your Formatter implementation is thread-safe.

Several Formatter implementations are provided in `format` subpackages as a convenience. The `number` package provides a `NumberFormatter`, `CurrencyFormatter`, and `PercentFormatter` to format `java.lang.Number` objects using a `java.text.NumberFormat`. The `datetime` package provides a `DateFormatter` to format `java.util.Date` objects with a `java.text.DateFormat`. The `datetime.joda` package provides comprehensive datetime formatting support based on the [Joda Time library](http://joda-time.sourceforge.net/).

Consider `DateFormatter` as an example `Formatter` implementation:

```java
package org.springframework.format.datetime;

public final class DateFormatter implements Formatter<Date> {

	private String pattern;

	public DateFormatter(String pattern) {
		this.pattern = pattern;
	}

	public String print(Date date, Locale locale) {
		if (date == null) {
			return "";
		}
		return getDateFormat(locale).format(date);
	}

	public Date parse(String formatted, Locale locale) throws ParseException {
		if (formatted.length() == 0) {
			return null;
		}
		return getDateFormat(locale).parse(formatted);
	}

	protected DateFormat getDateFormat(Locale locale) {
		DateFormat dateFormat = new SimpleDateFormat(this.pattern, locale);
		dateFormat.setLenient(false);
		return dateFormat;
	}

}
```

The Spring team welcomes community-driven `Formatter` contributions; see [jira.spring.io](https://jira.spring.io/browse/SPR) to contribute.

### 5.6.2 Annotation-driven Formatting

As you will see, field formatting can be configured by field type or annotation. To bind an Annotation to a formatter, implement AnnotationFormatterFactory:

```java
package org.springframework.format;

public interface AnnotationFormatterFactory<A extends Annotation> {

	Set<Class<?>> getFieldTypes();

	Printer<?> getPrinter(A annotation, Class<?> fieldType);

	Parser<?> getParser(A annotation, Class<?> fieldType);

}
```

Parameterize A to be the field annotationType you wish to associate formatting logic with, for example `org.springframework.format.annotation.DateTimeFormat`. Have `getFieldTypes()` return the types of fields the annotation may be used on. Have `getPrinter()` return a Printer to print the value of an annotated field. Have`getParser()` return a Parser to parse a clientValue for an annotated field.

The example AnnotationFormatterFactory implementation below binds the @NumberFormat Annotation to a formatter. This annotation allows either a number style or pattern to be specified:

```java
public final class NumberFormatAnnotationFormatterFactory
		implements AnnotationFormatterFactory<NumberFormat> {

	public Set<Class<?>> getFieldTypes() {
		return new HashSet<Class<?>>(asList(new Class<?>[] {
			Short.class, Integer.class, Long.class, Float.class,
			Double.class, BigDecimal.class, BigInteger.class }));
	}

	public Printer<Number> getPrinter(NumberFormat annotation, Class<?> fieldType) {
		return configureFormatterFrom(annotation, fieldType);
	}

	public Parser<Number> getParser(NumberFormat annotation, Class<?> fieldType) {
		return configureFormatterFrom(annotation, fieldType);
	}

	private Formatter<Number> configureFormatterFrom(NumberFormat annotation,
			Class<?> fieldType) {
		if (!annotation.pattern().isEmpty()) {
			return new NumberFormatter(annotation.pattern());
		} else {
			Style style = annotation.style();
			if (style == Style.PERCENT) {
				return new PercentFormatter();
			} else if (style == Style.CURRENCY) {
				return new CurrencyFormatter();
			} else {
				return new NumberFormatter();
			}
		}
	}
}
```

To trigger formatting, simply annotate fields with @NumberFormat:

```java
public class MyModel {

	@NumberFormat(style=Style.CURRENCY)
	private BigDecimal decimal;

}
```

#### Format Annotation API

A portable format annotation API exists in the `org.springframework.format.annotation` package. Use @NumberFormat to format java.lang.Number fields. Use @DateTimeFormat to format java.util.Date, java.util.Calendar, java.util.Long, or Joda Time fields.

The example below uses @DateTimeFormat to format a java.util.Date as a ISO Date (yyyy-MM-dd):

```java
public class MyModel {

	@DateTimeFormat(iso=ISO.DATE)
	private Date date;

}
```

### 5.6.3 FormatterRegistry SPI

The FormatterRegistry is an SPI for registering formatters and converters. `FormattingConversionService` is an implementation of FormatterRegistry suitable for most environments. This implementation may be configured programmatically or declaratively as a Spring bean using `FormattingConversionServiceFactoryBean`. Because this implementation also implements `ConversionService`, it can be directly configured for use with Spring’s DataBinder and the Spring Expression Language (SpEL).

Review the FormatterRegistry SPI below:

```java
package org.springframework.format;

public interface FormatterRegistry extends ConverterRegistry {

	void addFormatterForFieldType(Class<?> fieldType, Printer<?> printer, Parser<?> parser);

	void addFormatterForFieldType(Class<?> fieldType, Formatter<?> formatter);

	void addFormatterForFieldType(Formatter<?> formatter);

	void addFormatterForAnnotation(AnnotationFormatterFactory<?, ?> factory);

}
```

As shown above, Formatters can be registered by fieldType or annotation.

The FormatterRegistry SPI allows you to configure Formatting rules centrally, instead of duplicating such configuration across your Controllers. For example, you might want to enforce that all Date fields are formatted a certain way, or fields with a specific annotation are formatted in a certain way. With a shared FormatterRegistry, you define these rules once and they are applied whenever formatting is needed.

### 5.6.4 FormatterRegistrar SPI

The FormatterRegistrar is an SPI for registering formatters and converters through the FormatterRegistry:

```java
package org.springframework.format;

public interface FormatterRegistrar {

	void registerFormatters(FormatterRegistry registry);

}
```

A FormatterRegistrar is useful when registering multiple related converters and formatters for a given formatting category, such as Date formatting. It can also be useful where declarative registration is insufficient. For example when a formatter needs to be indexed under a specific field type different from its own <T> or when registering a Printer/Parser pair. The next section provides more information on converter and formatter registration.

### 5.6.5 Configuring Formatting in Spring MVC

See [Section 18.16.3, “Conversion and Formatting”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#mvc-config-conversion) in the Spring MVC chapter.