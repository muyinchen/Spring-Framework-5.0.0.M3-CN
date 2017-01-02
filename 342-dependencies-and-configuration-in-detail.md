### 3.4.2 Dependencies and configuration in detail `依赖和配置的种种细节`

As mentioned in the previous section, you can define bean properties and constructor arguments as references to other managed beans (collaborators), or as values defined inline. Spring’s XML-based configuration metadata supports sub-element types within its ` <property/>` and`<constructor-arg/>` elements for this purpose.

如上一节所述，您可以将bean属性和构造函数参数定义为对其他托管bean（协作者）的引用，或者作为内联定义的值。 Spring通过配置基于XML的元数据来支持它的`<property />`和`<constructor-arg />`元素中的子元素类型。

#### Straight values (primitives, Strings, and so on) ` 直值（基本类型，字符串等）`

The `value` attribute of the `<property/>` element specifies a property or constructor argument as a human-readable string representation. Spring’s [conversion service](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/#core-convert-ConversionService-API)is used to convert these  values from a `String` to the actual type of the property or argument.

`<property />`元素的`value`属性指定一个属性或构造函数参数作为一个人为可读的字符串表示。 Spring的[转换服务](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/#core-convert-ConversionService-API)用于将这些值从 一个`String`转换到属性或参数的实际类型。

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

##### The idref element `idref元素`

The `idref` element is simply an error-proof way to pass the *id* (string value - not a reference) of another bean in the container to a`<constructor-arg/>` or `<property/>` element.

`idref`元素只是一种将容器中另一个bean的* id *（字符串值 - 不是引用）传递给`<constructor-arg />`或`<property />`元素 。

```xml
<bean id="theTargetBean" class="..."/>

<bean id="theClientBean" class="...">
    <property name="targetName">
        <idref bean="theTargetBean" />
    </property>
</bean>
```

The above bean definition snippet is *exactly* equivalent (at runtime) to the following snippet:
上面的bean定义代码片段与下面的代码片段`完全相同`（在运行时）：

```xml
<bean id="theTargetBean" class="..." />

<bean id="client" class="...">
    <property name="targetName" value="theTargetBean" />
</bean>
```

The first form is preferable to the second, because using the `idref` tag allows the container to validate *at deployment time* that the referenced, named bean actually exists. In the second variation, no validation is performed on the value that is passed to the `targetName` property of the `client` bean. Typos are only discovered (with most likely fatal results) when the `client` bean is actually instantiated. If the `client` bean is a [prototype](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/#beans-factory-scopes) bean, this typo and the resulting exception may only be discovered long after the container is deployed.

| ![[Note]](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/images/note.png) |
| ---------------------------------------- |
| The `local` attribute on the `idref` element is no longer supported in the 4.0 beans xsd since it does not provide value over a regular `bean` reference anymore. Simply change your existing `idref local` references to `idref bean` when upgrading to the 4.0 schema. |

A common place (at least in versions earlier than Spring 2.0) where the `<idref/>` element brings value is in the configuration of [AOP interceptors](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/#aop-pfb-1) in a`ProxyFactoryBean` bean definition. Using `<idref/>` elements when you specify the interceptor names prevents you from misspelling an interceptor id.

第一种形式优于第二种形式，因为使用`idref`标签允许容器`在部署时`验证被引用的命名bean实际存在。在第二个变体中，不对传递给`client` bean的`targetName`属性的值执行验证。当`client` bean实际被实例化时，只有发现了typos（而且可能是致命的结果）。如果`client` bean是一个[prototype](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/#beans-factory-scopes) bean，这个打印错误所生成的异常只能在容器部署后很久才被发现。

| ！[[Note]](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/images/note.png) |
| ---------------------------------------- |
| `idref`元素上的`local`属性在4.0 bean xsd中不再支持，因为它不再提供超过正则`bean`引用的值。在升级到4.0模式时，只需将现有的“idref local”引用更改为“idref bean”。 |

`<idref />`元素带来的一个常见的价值（至少在Spring 2.0之前的版本中）是在[AOP拦截器](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/#aop-pfb-1) 的配置中。在一个`ProxyFactoryBean` bean定义中，当指定拦截器名称时，使用`<idref />`元素可防止拼写拦截器ID。

#### References to other beans (collaborators)`引用其他bean（协作者）`


The `ref` element is the final element inside a `<constructor-arg/>` or `<property/>` definition element. Here you set the value of the specified property of a bean to be a reference to another bean (a collaborator) managed by the container. The referenced bean is a dependency of the bean whose property will be set, and it is initialized on demand as needed before the property is set. (If the collaborator is a singleton bean, it may be initialized already by the container.) All references are ultimately a reference to another object. Scoping and validation depend on whether you specify the id/name of the other object through the `bean`, `local,` or `parent`attributes.

Specifying the target bean through the `bean` attribute of the `<ref/>` tag is the most general form, and allows creation of a reference to any bean in the same container or parent container, regardless of whether it is in the same XML file. The value of the `bean` attribute may be the same as the `id` attribute of the target bean, or as one of the values in the `name` attribute of the target bean.

`ref`元素是`<constructor-arg/>`或`<property/>`定义元素中的最后一个元素。在这里，你将bean的指定属性的值设置为对容器管理的另一个bean（协作者）的引用。引用的bean是将设置其属性的bean的依赖关系，并且在设置属性之前根据需要对其进行初始化。 （如果协作者是单例bean，它可能已经由容器初始化。）所有引用最终都是对另一个对象的引用。范围和验证取决于您是通过“bean”，“local”还是“parent”属性指定其他对象的id / name。

通过`<ref/>`标签的`bean`属性指定目标bean是最通用的形式，允许创建对同一容器或父容器中的任何bean的引用，而不管它是否在同一个XML文件。 “bean”属性的值可以与目标bean的“id”属性相同，也可以与目标bean的“name”属性中的值之一相同。

```xml
<ref bean="someBean"/>
```
Specifying the target bean through the `parent` attribute creates a reference to a bean that is in a parent container of the current container. The value of the `parent`attribute may be the same as either the `id` attribute of the target bean, or one of the values in the `name` attribute of the target bean, and the target bean must be in a parent container of the current one. You use this bean reference variant mainly when you have a hierarchy of containers and you want to wrap an existing bean in a parent container with a proxy that will have the same name as the parent bean
.
通过`parent`属性指定目标bean将创建对当前容器的父容器中的bean的引用。 “parent”属性的值可以与目标bean的“id”属性相同，也可以与目标bean的“name”属性中的一个值相同，并且目标bean必须位于父容器中 的当前一个。 你使用这个bean引用变体主要是当你有一个容器的层次结构，并且你想用一个与父bean同名的代理在一个父容器中包装一个现有的bean。

```xml
<!-- in the parent context -->
<bean id="accountService" class="com.foo.SimpleAccountService">
	<!-- insert dependencies as required as here -->
</bean>
```

```xml
<!-- in the child (descendant) context -->
<bean id="accountService" <!-- bean name is the same as the parent bean -->
	class="org.springframework.aop.framework.ProxyFactoryBean">
	<property name="target">
		<ref parent="accountService"/> <!-- notice how we refer to the parent bean -->
	</property>
	<!-- insert other configuration and dependencies as required here -->
</bean>
```

| ![[Note]](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| ---------------------------------------- |
|'ref`元素的`local`属性在4.0 beans xsd中不再支持，因为它不再提供超过正则范围`bean`引用的值。 在升级到4.0模式时，只需将现有的`ref local`引用更改为`ref bean` |

#### Inner beans

A `<bean/>` element inside the `<property/>` or `<constructor-arg/>` elements defines a so-called *inner bean*.
元素内的`<property />`或`<constructor-arg />`元素里面定义了一个所谓的*inner(内部)bean *。
```xml
<bean id="outer" class="...">
	<!-- instead of using a reference to a target bean, simply define the target bean inline -->
	<property name="target">
		<bean class="com.example.Person"> <!-- this is the inner bean -->
			<property name="name" value="Fiona Apple"/>
			<property name="age" value="25"/>
		</bean>
	</property>
</bean>
```


An inner bean definition does not require a defined id or name; if specified, the container does not use such a value as an identifier. The container also ignores the `scope` flag on creation: Inner beans are *always* anonymous and they are *always* created with the outer bean. It is *not* possible to inject inner beans into collaborating beans other than into the enclosing bean or to access them independently.

As a corner case, it is possible to receive destruction callbacks from a custom scope, e.g. for a request-scoped inner bean contained within a singleton bean: The creation of the inner bean instance will be tied to its containing bean, but destruction callbacks allow it to participate in the request scope’s lifecycle. This is not a common scenario; inner beans typically simply share their containing bean’s scope.

内部bean定义不需要定义的id或名称; 如果指定，容器不使用这样的值作为标识符。 容器在创建时也忽略`scope`标志：内部bean *总是*匿名的，它们总是用外部bean创建。 将内部bean注入到协作bean中而不是注入到封闭bean中或者独立访问它们是不可能的。

作为一种角落情况，可以从自定义范围接收销毁回调，例如。 对于包含在单例bean内的请求范围内部bean：内部bean实例的创建将绑定到其包含的bean，但销毁回调允许它参与请求范围的生命周期。 这不是一个常见的情况; 内部bean通常简单地共享它们被包含bean的范围。
