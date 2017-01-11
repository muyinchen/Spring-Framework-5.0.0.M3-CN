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

#### Collections `集合`

In the `<list/>`, `<set/>`, `<map/>`, and `<props/>` elements, you set the properties and arguments of the Java `Collection` types `List`, `Set`, `Map`, and `Properties`, respectively.
在`<list/>`，`<set/>`，`<map/>`和`<props/>`元素中，设置Java`Collection`类型`List`，`Set`，`Map`和`Properties`的属性和参数。

```xml
<bean id="moreComplexObject" class="example.ComplexObject">
	<!-- results in a setAdminEmails(java.util.Properties) call -->
	<property name="adminEmails">
		<props>
			<prop key="administrator">administrator@example.org</prop>
			<prop key="support">support@example.org</prop>
			<prop key="development">development@example.org</prop>
		</props>
	</property>
	<!-- results in a setSomeList(java.util.List) call -->
	<property name="someList">
		<list>
			<value>a list element followed by a reference</value>
			<ref bean="myDataSource" />
		</list>
	</property>
	<!-- results in a setSomeMap(java.util.Map) call -->
	<property name="someMap">
		<map>
			<entry key="an entry" value="just some string"/>
			<entry key ="a ref" value-ref="myDataSource"/>
		</map>
	</property>
	<!-- results in a setSomeSet(java.util.Set) call -->
	<property name="someSet">
		<set>
			<value>just some string</value>
			<ref bean="myDataSource" />
		</set>
	</property>
</bean>
```

*The value of a map key or value, or a set value, can also again be any of the following elements:*

*一个map键或值或set值也可以是以下任何元素：*
```properties
bean | ref | idref | list | set | map | props | value | null
```

##### Collection merging `集合合并`

The Spring container also supports the *merging* of collections. An application developer can define a parent-style `<list/>`, `<set/>`, `<map/>` or `<props/>` element, and have child-style `<list/>`, `<set/>`, `<map/>` or `<props/>` elements inherit and override values from the parent collection. That is, the child collection’s values are the result of merging the elements of the parent and child collections, with the child’s collection elements overriding values specified in the parent collection.

*This section on merging discusses the parent-child bean mechanism. Readers unfamiliar with parent and child bean definitions may wish to read the relevant section before continuing.*

The following example demonstrates collection merging:
Spring容器还支持集合的*合并*。 一个应用程序开发人员可以定义一个父类型 `<list/>`, `<set/>`, `<map/>` 或者`<props/>`, `<list/>``<set/>`，`<map/>`或`<props/>`元素继承并覆盖父集合中的值。 也就是说，子集合的值是合并父和子集合的元素的结果，子集合元素覆盖父集合中指定的值。

*本节关于合并讨论了父子bean的机制。 不熟悉父和子bean定义的读者可能希望在继续之前阅读相关章节。

以下示例演示集合合并：
```xml
<beans>
	<bean id="parent" abstract="true" class="example.ComplexObject">
		<property name="adminEmails">
			<props>
				<prop key="administrator">administrator@example.com</prop>
				<prop key="support">support@example.com</prop>
			</props>
		</property>
	</bean>
	<bean id="child" parent="parent">
		<property name="adminEmails">
			<!-- the merge is specified on the child collection definition -->
			<props merge="true">
				<prop key="sales">sales@example.com</prop>
				<prop key="support">support@example.co.uk</prop>
			</props>
		</property>
	</bean>
<beans>
```

Notice the use of the `merge=true` attribute on the `<props/>` element of the `adminEmails` property of the `child` bean definition. When the `child` bean is resolved and instantiated by the container, the resulting instance has an `adminEmails` `Properties` collection that contains the result of the merging of the child’s `adminEmails`collection with the parent’s `adminEmails` collection.

注意在`child` bean定义的`adminEmails`属性的`<props />`元素上使用`merge = true`属性。 当`child` bean被容器解析和实例化时，生成的实例会有一个`adminEmails``Properties`集合，其中包含子集`adminEmails`collection与父集合'adminEmails`集合的合并结果。
```properties
administrator=administrator@example.com
sales=sales@example.com
support=support@example.co.uk
```
The child `Properties` collection’s value set inherits all property elements from the parent `<props/>`, and the child’s value for the `support` value overrides the value in the parent collection.

This merging behavior applies similarly to the `<list/>`, `<map/>`, and `<set/>` collection types. In the specific case of the `<list/>` element, the semantics associated with the `List` collection type, that is, the notion of an `ordered` collection of values, is maintained; the parent’s values precede all of the child list’s values. In the case of the `Map`, `Set`, and `Properties` collection types, no ordering exists. Hence no ordering semantics are in effect for the collection types that underlie the associated `Map`, `Set`, and `Properties` implementation types that the container uses internally.

子属性`Properties`集合的值集合从父`<props/>`继承所有属性元素，`support`值的子值将覆盖父集合中的值。

这种合并行为类似地适用于`<list/>`，`<map/>`和`<set/>`集合类型。 在`<list/>`元素的特定情况下，与`List`集合类型相关联的语义，即`ordered`集合的值的概念被维护; 父级的值在所有子级列表的值之前。 在`Map`，`Set`和`Propertie`集合类型的情况下，不存在排序。 因此没有排序语义对集合类型有效，这些类型是容器在内部使用的相关联的`Map`，`Set`和`Properties`实现类型的基础。

##### Limitations of collection merging `集合合并的限制`

You cannot merge different collection types (such as a `Map` and a `List`), and if you do attempt to do so an appropriate `Exception` is thrown. The `merge` attribute must be specified on the lower, inherited, child definition; specifying the `merge` attribute on a parent collection definition is redundant and will not result in the desired merging.

你不能合并不同的集合类型（例如一个`Map`和一个`List`），如果你试图这样做，一个适当的`Exception`被抛出。 `merge`属性必须在下层，继承，子定义上指定; 在父集合定义上指定`merge`属性是多余的，并且不会导致所需的合并。

##### Strongly-typed collection `强类型集合`

With the introduction of generic types in Java 5, you can use strongly typed collections. That is, it is possible to declare a `Collection` type such that it can only contain`String` elements (for example). If you are using Spring to dependency-inject a strongly-typed `Collection` into a bean, you can take advantage of Spring’s type-conversion support such that the elements of your strongly-typed `Collection` instances are converted to the appropriate type prior to being added to the `Collection`.

随着在Java 5中引入泛型类型，可以使用强类型集合。 也就是说，可以声明一个`Collection`类型，使得它只能包含`String`元素（例如）。 如果你使用Spring来将一个强类型的“Collection”依赖注入到一个bean中，你可以利用Spring的类型转换支持，这样强类型的“Collection”实例的元素被转换为适当的类型 被添加到`Collection`。

```java
public class Foo {

	private Map<String, Float> accounts;

	public void setAccounts(Map<String, Float> accounts) {
		this.accounts = accounts;
	}
}
```

```xml
<beans>
	<bean id="foo" class="x.y.Foo">
		<property name="accounts">
			<map>
				<entry key="one" value="9.99"/>
				<entry key="two" value="2.75"/>
				<entry key="six" value="3.99"/>
			</map>
		</property>
	</bean>
</beans>
```

When the `accounts` property of the `foo` bean is prepared for injection, the generics information about the element type of the strongly-typed `Map` is available by reflection. Thus Spring’s type conversion infrastructure recognizes the various value elements as being of type `Float`, and the string values `9.99, 2.75`, and `3.99` are converted into an actual `Float` type.

当`foo` bean的`accounts`属性准备注入时，强类型`Map`的元素类型的泛型信息可以通过反射来获得。 因此，Spring的类型转换基础设施将各种值元素识别为“Float”类型，并且字符串值“9.99,2.75”和“3.99”被转换为实际的“Float”类型。

#### References to other beans (collaborators) `引用其他bean（协作者）`

The `ref` element is the final element inside a `<constructor-arg/>` or `<property/>` definition element. Here you set the value of the specified property of a bean to be a reference to another bean (a collaborator) managed by the container. The referenced bean is a dependency of the bean whose property will be set, and it is initialized on demand as needed before the property is set. (If the collaborator is a singleton bean, it may be initialized already by the container.) All references are ultimately a reference to another object. Scoping and validation depend on whether you specify the id/name of the other object through the `bean`, `local,` or `parent`attributes.

Specifying the target bean through the `bean` attribute of the `<ref/>` tag is the most general form, and allows creation of a reference to any bean in the same container or parent container, regardless of whether it is in the same XML file. The value of the `bean` attribute may be the same as the `id` attribute of the target bean, or as one of the values in the `name` attribute of the target bean.

`ref`元素是`<constructor-arg/>`或`<property/>`定义元素中的最后一个元素。在这里，您将bean的指定属性的值设置为对容器管理的另一个bean（协作者）的引用。该引用bean将被作为依赖注入，并且在注入属性之前根据需要对其进行初始化。 （如果协作者是单例bean，它可能已经由容器初始化）。所有引用最终都是对另一个对象的引用。范围和验证取决于你是通过`bean`，`local`还是`parent`属性指定其他对象的id / name。

通过`<ref/>`标签的`bean`属性指定目标bean是最常见的形式，允许创建对同一容器或父容器中的任何bean的引用，而不管它是否在同一个XML文件。`bean`属性的值可以与目标bean的`id`属性相同，也可以与目标bean的`name`属性中的值之一相同。
```xml
<ref bean="someBean"/>
```
Specifying the target bean through the `parent` attribute creates a reference to a bean that is in a parent container of the current container. The value of the `parent`attribute may be the same as either the `id` attribute of the target bean, or one of the values in the `name` attribute of the target bean, and the target bean must be in a parent container of the current one. You use this bean reference variant mainly when you have a hierarchy of containers and you want to wrap an existing bean in a parent container with a proxy that will have the same name as the parent bean.

通过`parent`属性指定目标bean将创建对当前容器的父容器中的bean的引用。`parent`属性的值可以与目标bean的`id`属性相同，也可以与目标bean的`name`属性中的一个值相同，并且目标bean必须位于父容器中 的当前一个。 你使用这个bean引用变体主要是当你有一个容器的层次结构，并且你想用一个与父bean同名的代理在一个父容器中包装一个现有的bean。
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
| `ref`元素的`local`属性在4.0 beans xsd中不再支持，因为它不再提供超过正则`bean`引用的值。 在升级到4.0模式时，只需将现有的`ref local`引用更改为`ref bean`. |

#### Inner beans `内部bean`

A `<bean/>` element inside the `<property/>` or`<constructor-arg/>` elements defines a so-called *inner bean*.
通过元素`<bean/>` 在`<property/>`或`<constructor-arg/>`元素内定义了一个所谓的*内部bean *。
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

内部bean定义不需要定义的id或名称; 如果指定，容器不使用这样的值作为标识符。 容器在创建时忽略`scope`标志：内部bean *总是*匿名的，它们总是用外部bean创建。 将内部bean注入到协作bean中而不是注入到封装bean中或者独立访问它们是不可能的。

作为一种角落情况，可以从自定义范围接收销毁回调，例如。 对于包含在单例bean内的请求范围内部bean：内部bean实例的创建将绑定到其包含的bean，但销毁回调允许它参与请求范围的生命周期。 这不是一个常见的情况; 内部bean通常简单地共享它们的包含bean的范围。


