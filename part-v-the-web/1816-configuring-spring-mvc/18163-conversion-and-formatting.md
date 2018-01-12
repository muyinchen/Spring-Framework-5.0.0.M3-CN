### 18.16.3转换和格式化

默认情况下，`Number`和`Date`类型的格式化程序已安装，包括支持`@NumberFormat`和`@DateTimeFormat`注解。 如果Joda时间存在于类路径中，则还会安装对Joda时间格式库的完全支持。 要注册自定义格式器和转换器，请重写`addFormatters`方法：

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

在MVC命名空间中，添加`<mvc:annotation-driven>`时会应用相同的默认值。 要注册自定义格式化器和转换器，只需提供一个`ConversionService`：

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
| 有关何时使用FormatterRegistrars的更多信息，请参见[Section5.6.4, “FormatterRegistrar SPI”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/validation.html#format-FormatterRegistrar-SPI)和`FormattingConversionServiceFactoryBean`。 |



