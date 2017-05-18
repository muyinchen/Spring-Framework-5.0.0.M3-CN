## 5.5 Spring Type Conversion

Spring 3 introduces a `core.convert` package that provides a general type conversion system. The system defines an SPI to implement type conversion logic, as well as an API to execute type conversions at runtime. Within a Spring container, this system can be used as an alternative to PropertyEditors to convert externalized bean property value strings to required property types. The public API may also be used anywhere in your application where type conversion is needed.

### 5.5.1 Converter SPI

The SPI to implement type conversion logic is simple and strongly typed:

```java
package org.springframework.core.convert.converter;

public interface Converter<S, T> {

	T convert(S source);

}
```

To create your own converter, simply implement the interface above. Parameterize `S` as the type you are converting from, and `T` as the type you are converting to. Such a converter can also be applied transparently if a collection or array of `S` needs to be converted to an array or collection of `T`, provided that a delegating array/collection converter has been registered as well (which `DefaultConversionService` does by default).

For each call to `convert(S)`, the source argument is guaranteed to be NOT null. Your Converter may throw any unchecked exception if conversion fails; specifically, an`IllegalArgumentException` should be thrown to report an invalid source value. Take care to ensure that your `Converter` implementation is thread-safe.

Several converter implementations are provided in the `core.convert.support` package as a convenience. These include converters from Strings to Numbers and other common types. Consider `StringToInteger` as an example for a typical `Converter` implementation:

```java
package org.springframework.core.convert.support;

final class StringToInteger implements Converter<String, Integer> {

	public Integer convert(String source) {
		return Integer.valueOf(source);
	}

}
```

### 5.5.2 ConverterFactory

When you need to centralize the conversion logic for an entire class hierarchy, for example, when converting from String to java.lang.Enum objects, implement`ConverterFactory`:

```java
package org.springframework.core.convert.converter;

public interface ConverterFactory<S, R> {

	<T extends R> Converter<S, T> getConverter(Class<T> targetType);

}
```

Parameterize S to be the type you are converting from and R to be the base type defining the *range* of classes you can convert to. Then implement getConverter(Class<T>), where T is a subclass of R.

Consider the `StringToEnum` ConverterFactory as an example:

```java
package org.springframework.core.convert.support;

final class StringToEnumConverterFactory implements ConverterFactory<String, Enum> {

	public <T extends Enum> Converter<String, T> getConverter(Class<T> targetType) {
		return new StringToEnumConverter(targetType);
	}

	private final class StringToEnumConverter<T extends Enum> implements Converter<String, T> {

		private Class<T> enumType;

		public StringToEnumConverter(Class<T> enumType) {
			this.enumType = enumType;
		}

		public T convert(String source) {
			return (T) Enum.valueOf(this.enumType, source.trim());
		}
	}
}
```

### 5.5.3 GenericConverter

When you require a sophisticated Converter implementation, consider the GenericConverter interface. With a more flexible but less strongly typed signature, a GenericConverter supports converting between multiple source and target types. In addition, a GenericConverter makes available source and target field context you can use when implementing your conversion logic. Such context allows a type conversion to be driven by a field annotation, or generic information declared on a field signature.

```java
package org.springframework.core.convert.converter;

public interface GenericConverter {

	public Set<ConvertiblePair> getConvertibleTypes();

	Object convert(Object source, TypeDescriptor sourceType, TypeDescriptor targetType);

}
```

To implement a GenericConverter, have getConvertibleTypes() return the supported source→target type pairs. Then implement convert(Object, TypeDescriptor, TypeDescriptor) to implement your conversion logic. The source TypeDescriptor provides access to the source field holding the value being converted. The target TypeDescriptor provides access to the target field where the converted value will be set.

A good example of a GenericConverter is a converter that converts between a Java Array and a Collection. Such an ArrayToCollectionConverter introspects the field that declares the target Collection type to resolve the Collection’s element type. This allows each element in the source array to be converted to the Collection element type before the Collection is set on the target field.

| ![[Note]](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| ---------------------------------------- |
| Because GenericConverter is a more complex SPI interface, only use it when you need it. Favor Converter or ConverterFactory for basic type conversion needs. |

#### ConditionalGenericConverter

Sometimes you only want a `Converter` to execute if a specific condition holds true. For example, you might only want to execute a `Converter` if a specific annotation is present on the target field. Or you might only want to execute a `Converter` if a specific method, such as a `static valueOf` method, is defined on the target class.`ConditionalGenericConverter` is the union of the `GenericConverter` and `ConditionalConverter` interfaces that allows you to define such custom matching criteria:

```java
public interface ConditionalGenericConverter
        extends GenericConverter, ConditionalConverter {

	boolean matches(TypeDescriptor sourceType, TypeDescriptor targetType);

}
```

A good example of a `ConditionalGenericConverter` is an EntityConverter that converts between an persistent entity identifier and an entity reference. Such a EntityConverter might only match if the target entity type declares a static finder method e.g. `findAccount(Long)`. You would perform such a finder method check in the implementation of `matches(TypeDescriptor, TypeDescriptor)`.

### 5.5.4 ConversionService API

The ConversionService defines a unified API for executing type conversion logic at runtime. Converters are often executed behind this facade interface:

```java
package org.springframework.core.convert;

public interface ConversionService {

	boolean canConvert(Class<?> sourceType, Class<?> targetType);

	<T> T convert(Object source, Class<T> targetType);

	boolean canConvert(TypeDescriptor sourceType, TypeDescriptor targetType);

	Object convert(Object source, TypeDescriptor sourceType, TypeDescriptor targetType);

}
```

Most ConversionService implementations also implement `ConverterRegistry`, which provides an SPI for registering converters. Internally, a ConversionService implementation delegates to its registered converters to carry out type conversion logic.

A robust ConversionService implementation is provided in the `core.convert.support` package. `GenericConversionService` is the general-purpose implementation suitable for use in most environments. `ConversionServiceFactory` provides a convenient factory for creating common ConversionService configurations.

### 5.5.5 Configuring a ConversionService

A ConversionService is a stateless object designed to be instantiated at application startup, then shared between multiple threads. In a Spring application, you typically configure a ConversionService instance per Spring container (or ApplicationContext). That ConversionService will be picked up by Spring and then used whenever a type conversion needs to be performed by the framework. You may also inject this ConversionService into any of your beans and invoke it directly.

| ![[Note]](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| ---------------------------------------- |
| If no ConversionService is registered with Spring, the original PropertyEditor-based system is used. |

To register a default ConversionService with Spring, add the following bean definition with id `conversionService`:

```xml
<bean id="conversionService"
	class="org.springframework.context.support.ConversionServiceFactoryBean"/>
```

A default ConversionService can convert between strings, numbers, enums, collections, maps, and other common types. To supplement or override the default converters with your own custom converter(s), set the `converters` property. Property values may implement either of the Converter, ConverterFactory, or GenericConverter interfaces.

```xml
<bean id="conversionService"
		class="org.springframework.context.support.ConversionServiceFactoryBean">
	<property name="converters">
		<set>
			<bean class="example.MyCustomConverter"/>
		</set>
	</property>
</bean>
```

It is also common to use a ConversionService within a Spring MVC application. See [Section 18.16.3, “Conversion and Formatting”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#mvc-config-conversion) in the Spring MVC chapter.

In certain situations you may wish to apply formatting during conversion. See [Section 5.6.3, “FormatterRegistry SPI”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#format-FormatterRegistry-SPI) for details on using`FormattingConversionServiceFactoryBean`.

### 5.5.6 Using a ConversionService programmatically

To work with a ConversionService instance programmatically, simply inject a reference to it like you would for any other bean:

```java
@Service
public class MyService {

	@Autowired
	public MyService(ConversionService conversionService) {
		this.conversionService = conversionService;
	}

	public void doIt() {
		this.conversionService.convert(...)
	}
}
```

For most use cases, the `convert` method specifying the *targetType* can be used but it will not work with more complex types such as a collection of a parameterized element. If you want to convert a `List` of `Integer` to a `List` of `String` programmatically, for instance, you need to provide a formal definition of the source and target types.

Fortunately, `TypeDescriptor` provides various options to make that straightforward:

```java
DefaultConversionService cs = new DefaultConversionService();

List<Integer> input = ....
cs.convert(input,
	TypeDescriptor.forObject(input), // List<Integer> type descriptor
	TypeDescriptor.collection(List.class, TypeDescriptor.valueOf(String.class)));
```

Note that `DefaultConversionService` registers converters automatically which are appropriate for most environments. This includes collection converters, scalar converters, and also basic `Object` to `String` converters. The same converters can be registered with any `ConverterRegistry` using the *static*`addDefaultConverters` method on the `DefaultConversionService` class.

Converters for value types will be reused for arrays and collections, so there is no need to create a specific converter to convert from a `Collection` of `S` to a`Collection` of `T`, assuming that standard collection handling is appropriate.