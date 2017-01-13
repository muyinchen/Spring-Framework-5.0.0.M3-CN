### 3.4.5 Autowiring collaborators `自动装配协作者`

The Spring container can *autowire* relationships between collaborating beans. You can allow Spring to resolve collaborators (other beans) automatically for your bean by inspecting the contents of the `ApplicationContext`. Autowiring has the following advantages:

- Autowiring can significantly reduce the need to specify properties or constructor arguments. (Other mechanisms such as a bean template [discussed elsewhere in this chapter](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/#beans-child-bean-definitions) are also valuable in this regard.)
- Autowiring can update a configuration as your objects evolve. For example, if you need to add a dependency to a class, that dependency can be satisfied automatically without you needing to modify the configuration. Thus autowiring can be especially useful during development, without negating the option of switching to explicit wiring when the code base becomes more stable.

When using XML-based configuration metadata [[2\]](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/#ftn.d5e1508), you specify autowire mode for a bean definition with the `autowire` attribute of the `<bean/>` element. The autowiring functionality has four modes. You specify autowiring *per* bean and thus can choose which ones to autowire.

Spring容器可以*自动装配*协作bean之间的关系。你可以允许Spring通过检查`ApplicationContext`的内容来自动为你的bean解析协作者（其他bean）。自动装配有以下优点：

- 自动装配可以显着减少指定属性或构造函数参数的需求.（其他机制，如bean模板[在本章其他地方讨论](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/#beans-child-bean-definitions) 在这方面也很有价值。）
- 自动装配可以在您的对象发生变化时更新配置。例如，如果您需要向类添加依赖关系，则可以自动满足该依赖关系，而无需修改配置。因此，自动依赖在开发期间可以是特别有用的，当系统趋于稳定时改为显式装配。

当使用基于XML的配置元数据[[2 \]](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/#ftn.d5e1508)时，您指定autowire“属性的bean定义的自动装配模式。自动装配功能有四种模式。您指定自动装配*每个*bean，因此可以选择哪些自动装配。