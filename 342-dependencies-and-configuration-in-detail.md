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


The preceding XML is more succinct; however, typos are discovered at runtime rather than design time, unless you use an IDE such as [IntelliJ IDEA](http://www.jetbrains.com/idea/) or the [Spring Tool Suite](https://spring.io/tools/sts) (STS) that support automatic property completion when you create bean definitions. Such IDE assistance is highly recommended.

You can also configure a `java.util.Properties` instance as:

前面的XML更简洁; 但是，除非您使用[IntelliJ IDEA](http://www.jetbrains.com/idea/)或[Spring Tool Suite](https://spring.io/tools/sts) (STS)等IDE，否则会在运行时而不是设计时间(也就是在敲代码的时候)发现打字错误，在创建bean定义时支持自动属性完成。 强烈推荐此类IDE帮助。

您还可以将`java.util.Properties`实例配置为：

```xml
<bean id="mappings"
    class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">

    <!-- typed as a java.util.Properties -->
    <property name="properties">
        <value>
            jdbc.driver.className=com.mysql.jdbc.Driver
            jdbc.url=jdbc:mysql://localhost:3306/mydb
        </value>
    </property>
</bean>
```

The Spring container converts the text inside the `<value/>` element into a `java.util.Properties` instance by using the JavaBeans `PropertyEditor` mechanism. This is a nice shortcut, and is one of a few places where the Spring team do favor the use of the nested `<value/>` element over the `value` attribute style.

Spring容器通过使用JavaBeans的PropertyEditor机制将`<value />`元素内的文本转换为`java.util.Properties`实例。 这是一个很好的快捷方式，并且是Spring团队喜欢在“value”属性样式上使用嵌套的`<value />`元素的几个地方之一。
