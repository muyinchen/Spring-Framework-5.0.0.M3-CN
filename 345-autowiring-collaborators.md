### 3.4.5 Autowiring collaborators `自动装配协作者`

The Spring container can *autowire* relationships between collaborating beans. You can allow Spring to resolve collaborators (other beans) automatically for your bean by inspecting the contents of the `ApplicationContext`. Autowiring has the following advantages:

- Autowiring can significantly reduce the need to specify properties or constructor arguments. (Other mechanisms such as a bean template [discussed elsewhere in this chapter](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/#beans-child-bean-definitions) are also valuable in this regard.)
- Autowiring can update a configuration as your objects evolve. For example, if you need to add a dependency to a class, that dependency can be satisfied automatically without you needing to modify the configuration. Thus autowiring can be especially useful during development, without negating the option of switching to explicit wiring when the code base becomes more stable.

When using XML-based configuration metadata [[2\]](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/#ftn.d5e1508), you specify autowire mode for a bean definition with the `autowire` attribute of the `<bean/>` element. The autowiring functionality has four modes. You specify autowiring *per* bean and thus can choose which ones to autowire.

Spring容器可以*自动装配*协作bean之间的关系。你可以允许Spring通过检查`ApplicationContext`的内容来自动为你的bean解析协作者（其他bean）。自动装配有以下优点：

- 自动装配可以显着减少指定属性或构造函数参数的需求.（其他机制，如bean模板[在本章其他地方讨论](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/#beans-child-bean-definitions) 在这方面也很有价值。）
- 自动装配可以在您的对象发生变化时更新配置。例如，如果您需要向类添加依赖关系，则可以自动满足该依赖关系，而无需修改配置。因此，自动依赖在开发期间可以是特别有用的，当系统趋于稳定时改为显式装配。

当使用基于XML的配置元数据[[2 \]](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/#ftn.d5e1508)时，您指定autowire“属性的bean定义的自动装配模式。自动装配功能有四种模式。您指定自动装配*每个*bean，因此可以选择哪些自动装配。


**Table 3.2. Autowiring modes**

| Mode        | Explanation  模式解释                            |
| ----------- | ---------------------------------------- |
| no          | （默认）无自动装配。 Bean引用必须通过`ref`元素定义。 对于较大型部署，不建议更改默认设置，因为明确指定协作者会提供更好控制和清晰度。 在一定程度上，它记录了系统的结构. |
| byName      |按属性名称自动装配。 Spring查找与需要自动注入的属性同名的bean。 例如，如果bean定义设置为autowire by name，并且它包含* master *属性（即它有一个* setMaster（..）*方法），Spring会查找名为`master`的bean定义， 并使用它来设置属性. |
| byType      |允许属性在属性类型中只有一个bean存在于容器中时自动连接。 如果存在多个，则会抛出致命异常，这表示您不能对该bean使用* byType * autowiring。 如果没有匹配的bean，什么都不发生; 该属性未设置. |
| constructor |类似于* byType *，但适用于构造函数参数。如果在容器中没有找到与构造器参数类型一致的bean，则会抛出异常. |

With *byType* or *constructor* autowiring mode, you can wire arrays and typed-collections. In such cases *all* autowire candidates within the container that match the expected type are provided to satisfy the dependency. You can autowire strongly-typed Maps if the expected key type is `String`. An autowired Maps values will consist of all bean instances that match the expected type, and the Maps keys will contain the corresponding bean names.

You can combine autowire behavior with dependency checking, which is performed after autowiring completes.

使用* byType *或*构造函数*自动装配模式，可以应用于数组和类型集合。 在这种情况下，提供容器中所有匹配的自动装配对象将 被应用于满足各种依赖。 如果预期的键类型为“String”，则可以自动注入强类型的Map。一个自动装配的Map value值将包含与预期类型匹配的所有Bean实例。

你可以结合自动装配和依赖检查，后者将会在自动装配完成之后进行。
