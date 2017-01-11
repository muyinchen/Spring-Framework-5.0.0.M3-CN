### 3.4.3 Using depends-on

If a bean is a dependency of another that usually means that one bean is set as a property of another. Typically you accomplish this with the [`<ref/>` element](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/#beans-ref-element) in XML-based configuration metadata. However, sometimes dependencies between beans are less direct; for example, a static initializer in a class needs to be triggered, such as database driver registration. The `depends-on` attribute can explicitly force one or more beans to be initialized before the bean using this element is initialized. The following example uses the `depends-on` attribute to express a dependency on a single bean:

如果bean是另一个的依赖，通常意味着一个bean被设置为另一个的属性。 通常你用[`<ref />`element](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/#beans-ref-element) 在XML中配置依赖。 然而，有时bean之间的依赖不直接; 例如，类的静态块初始化，又比如数据库驱动程序注册。  `depends-on `属性可以在使用此元素的bean初始化之前明确强制一个或多个bean被初始化。 以下示例使用 `depends-on`属性来表示对单个bean的依赖关系：

```xml
<bean id="beanOne" class="ExampleBean" depends-on="manager"/>
<bean id="manager" class="ManagerBean" />
```

为了实现多个bean的依赖，你可以在depends-on中将指定的多个bean名字用分隔符进行分隔，分隔符可以是逗号，空格以及分号等。

```xml
<bean id="beanOne" class="ExampleBean" depends-on="manager,accountDao">
	<property name="manager" ref="manager" />
</bean>

<bean id="manager" class="ManagerBean" />
<bean id="accountDao" class="x.y.jdbc.JdbcAccountDao" />
```

| ![[Note]](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| ---------------------------------------- |
|bean定义中的`depends-on`属性可以指定初始化时间依赖性，在[singleton](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/#beans-factory-scopes-singleton) 的情况下只有bean，一个相应的销毁时间依赖。 在给定的bean本身被销毁之前，首先销毁定义与给定bean的`依赖关系`关系的依赖bean。 因此，`依赖`也可以控制销毁的顺序. |