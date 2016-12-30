### 3.4.2 Dependencies and configuration in detail `依赖和配置的种种细节`

As mentioned in the previous section, you can define bean properties and constructor arguments as references to other managed beans (collaborators), or as values defined inline. Spring’s XML-based configuration metadata supports sub-element types within its ` <property/>` and`<constructor-arg/>` elements for this purpose.

如上一节所述，您可以将bean属性和构造函数参数定义为对其他托管bean（协作者）的引用，或者作为内联定义的值。 Spring通过配置基于XML的元数据来支持它的`<property />`和`<constructor-arg />`元素中的子元素类型。

#### Straight values (primitives, Strings, and so on) ` 直值（基本类型，字符串等）`

The `value` attribute of the `<property/>` element specifies a property or constructor argument as a human-readable string representation. Spring’s [conversion service](http://docs.spring.io/spring/docs/5.0.0.M3/spring-framework-reference/htmlsingle/#core-convert-ConversionService-API)is used to convert these  values from a `String` to the actual type of the property or argument.

`<property />`元素的`value`属性指定一个属性或构造函数参数作为一个人为可读的字符串表示。 Spring的[转换服务](http://docs.spring.io/spring/docs/5.0.0.M3/spring-framework-reference/htmlsingle/#core-convert-ConversionService-API)用于将这些值从 一个`String`转换到属性或参数的实际类型。

```xml
<bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
    <!-- results in a setDriverClassName(String) call -->
    <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql://localhost:3306/mydb"/>
    <property name="username" value="root"/>
    <property name="password" value="masterkaoli"/>
</bean>
```