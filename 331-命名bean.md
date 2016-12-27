### 3.3.1 命名bean

Every bean has one or more identifiers. These identifiers must be unique within the container that hosts the bean. A bean usually has only one identifier, but if it requires more than one, the extra ones can be considered aliases.

In XML-based configuration metadata, you use the `id` and/or `name` attributes to specify the bean identifier(s). The `id` attribute allows you to specify exactly one id. Conventionally these names are alphanumeric ('myBean', 'fooService', etc.), but may contain special characters as well. If you want to introduce other aliases to the bean, you can also specify them in the `name` attribute, separated by a comma (`,`), semicolon (`;`), or white space. As a historical note, in versions prior to Spring 3.1, the `id` attribute was defined as an `xsd:ID` type, which constrained possible characters. As of 3.1, it is defined as an `xsd:string` type. Note that bean `id` uniqueness is still enforced by the container, though no longer by XML parsers.

You are not required to supply a name or id for a bean. If no name or id is supplied explicitly, the container generates a unique name for that bean. However, if you want to refer to that bean by name, through the use of the `ref` element or [Service Locator](http://docs.spring.io/spring/docs/5.0.0.M3/spring-framework-reference/htmlsingle/#beans-servicelocator) style lookup, you must provide a name. Motivations for not supplying a name are related to using [inner beans](http://docs.spring.io/spring/docs/5.0.0.M3/spring-framework-reference/htmlsingle/#beans-inner-beans) and [autowiring collaborators](http://docs.spring.io/spring/docs/5.0.0.M3/spring-framework-reference/htmlsingle/#beans-factory-autowire).

每个bean都有一个或多个标识符。这些标识符在托管bean的容器中必须是唯一的。一个bean通常只有一个标识符，但是如果它需要多个标识符，那么额外的标识符可以被认为是别名。

在基于XML的配置元数据中，您使用`id`和/或`name`属性来指定bean标识符。 `id`属性允许你指定一个id。通常这些名称是字母数字的('myBean'，'fooService'等)，但也可能包含特殊字符。如果要向bean引入其他别名，还可以在`name`属性中指定它们，用逗号(`，`)，分号(`;`)或空格分隔。作为一个历史记录，在Spring 3.1之前的版本中，'id'属性被定义为一个`xsd：ID'类型，它限制了可能的字符。从3.1开始，它被定义为一个`xsd：string`类型。注意bean'id'唯一性仍然由容器强制执行，虽然不再由XML解析器。

您不需要为bean提供名称或ID。如果没有明确提供名称或ID，容器将为该bean生成一个唯一的名称。但是，如果你想通过名称引用那个bean，通过使用`ref`元素或[Service Locator](http://docs.spring.io/spring/docs/5.0.0.M3/spring- framework-reference / htmlsingle /＃beans-servicelocator)样式查找，您必须提供一个名称。不提供名称的动机与使用[内部bean](http://docs.spring.io/spring/docs/5.0.0.M3/spring-framework-reference/htmlsingle/#beans-inner-beans)和[自动装配协作者](http://docs.spring.io/spring/docs/5.0.0.M3/spring-framework-reference/htmlsingle/#beans-factory-autowire)相关。


**Bean命名约定**

The convention is to use the standard Java convention for instance field names when naming beans. That is, bean names start with a lowercase letter, and are camel-cased from then on. Examples of such names would be (without quotes) `'accountManager'`, `'accountService'`, `'userDao'`, `'loginController'`, and so forth.

Naming beans consistently makes your configuration easier to read and understand, and if you are using Spring AOP it helps a lot when applying advice to a set of beans related by name.

约定是在命名bean时使用标准Java约定作为实例字段名称。 也就是说，bean名称以小写字母开头，从那时开始是驼峰式的。 这样的名称的示例将是（无引号）`'accountManager'，`'accountService'`，''userDao'`，`'loginController''等。

命名Bean一致地使您的配置更容易阅读和理解，如果您使用Spring AOP，它对按照名称相关的一组bean应用建议时会有很多帮助。

| ![[Note]](http://docs.spring.io/spring/docs/5.0.0.M3/spring-framework-reference/htmlsingle/images/note.png) |
| ---------------------------------------- |
| With component scanning in the classpath, Spring generates bean names for unnamed components, following the rules above: essentially, taking the simple class name and turning its initial character to lower-case. However, in the (unusual) special case when there is more than one character and both the first and second characters are upper case, the original casing gets preserved. These are the same rules as defined by `java.beans.Introspector.decapitalize` (which Spring is using here).
通过类路径中的组件扫描，Spring根据上面的规则生成未命名组件的bean名称：基本上，取简单的类名称并将其初始字符转换为小写。 然而，在（异常）特殊情况下，当存在多个字符并且第一和第二字符都是大写字母时，原始形式被保留。 这些是由`java.beans.Introspector.decapitalize`（Spring在这里使用）定义的相同规则 |

#### Aliasing a bean outside the bean definition  `bean 的别名`

In a bean definition itself, you can supply more than one name for the bean, by using a combination of up to one name specified by the `id` attribute, and any number of other names in the `name` attribute. These names can be equivalent aliases to the same bean, and are useful for some situations, such as allowing each component in an application to refer to a common dependency by using a bean name that is specific to that component itself.

Specifying all aliases where the bean is actually defined is not always adequate, however. It is sometimes desirable to introduce an alias for a bean that is defined elsewhere. This is commonly the case in large systems where configuration is split amongst each subsystem, each subsystem having its own set of object definitions. In XML-based configuration metadata, you can use the `<alias/>` element to accomplish this.

在对 bean 定义时，可以通过使用由`id`属性指定指定一个唯一的名称外，为了提供多个名称，需要通过name 属性加以指定，所有这个名称都指向同一个bean，在某些情况下提供别名非常有用，例如为了让应用每一个组件都能更容易的对公共组件进行引用。

然而，指定bean实际定义的所有别名并不总是足够的。 有时需要为在其他地方定义的bean引入别名。 在大型系统中通常是这种情况，其中配置在每个子系统之间分割，每个子系统具有其自己的对象定义集合。 在基于XML的配置元数据中，您可以使用`<alias/>`元素来实现这一点。

```xml
<alias name="fromName" alias="toName"/>
```

In this case, a bean in the same container which is named `fromName`, may also, after the use of this alias definition, be referred to as `toName`.

For example, the configuration metadata for subsystem A may refer to a DataSource via the name `subsystemA-dataSource`. The configuration metadata for subsystem B may refer to a DataSource via the name `subsystemB-dataSource`. When composing the main application that uses both these subsystems the main application refers to the DataSource via the name `myApp-dataSource`. To have all three names refer to the same object you add to the MyApp configuration metadata the following aliases definitions:

在这种情况下，名为`fromName`的同一容器中的bean也可以在使用此别名定义之后称为`toName`。

例如，子系统A的配置元数据可以通过名称“subsystemA-dataSource”引用DataSource。 子系统B的配置元数据可以通过名称“subsystemB-dataSource”引用DataSource。 当编译使用这两个子系统的主应用程序时，主应用程序通过名称`myApp-dataSource`引用DataSource。 要使所有三个名称引用添加到MyApp配置元数据中的同一对象，以下别名定义：

```
<alias name="subsystemA-dataSource" alias="subsystemB-dataSource"/>
<alias name="subsystemA-dataSource" alias="myApp-dataSource" />
```

Now each component and the main application can refer to the dataSource through a name that is unique and guaranteed not to clash with any other definition (effectively creating a namespace), yet they refer to the same bean.
现在每个组件和主应用程序可以通过唯一的名称引用dataSource，并且保证不与任何其他定义（有效地创建命名空间）冲突，但它们引用同一个bean。

**基于 Java 的配置**

如果你使用Java配置，`@ Bean`注解可以用来提供别名,详细信息请看[Section 3.12.3, “Using the @Bean annotation”](http://docs.spring.io/spring/docs/5.0.0.M3/spring-framework-reference/htmlsingle/#beans-java-bean-annotation).
