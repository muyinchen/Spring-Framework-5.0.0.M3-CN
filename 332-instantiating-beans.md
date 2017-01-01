
### 3.3.2 Instantiating beans `实例化bean`

A bean definition essentially is a recipe for creating one or more objects. The container looks at the recipe for a named bean when asked, and uses the configuration metadata encapsulated by that bean definition to create (or acquire) an actual object.

If you use XML-based configuration metadata, you specify the type (or class) of object that is to be instantiated in the `class` attribute of the ` <bean/> ` element. This`class` attribute, which internally is a `Class` property on a `BeanDefinition` instance, is usually mandatory. (For exceptions, see [the section called “Instantiation using an instance factory method”](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/#beans-factory-class-instance-factory-method) and [Section 3.7, “Bean definition inheritance”](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/#beans-child-bean-definitions).) You use the `Class` property in one of two ways:

- Typically, to specify the bean class to be constructed in the case where the container itself directly creates the bean by calling its constructor reflectively, somewhat equivalent to Java code using the `new` operator.
- To specify the actual class containing the `static` factory method that will be invoked to create the object, in the less common case where the container invokes a`static` *factory* method on a class to create the bean. The object type returned from the invocation of the `static` factory method may be the same class or another class entirely.

bean定义本质上是创建一个或多个对象的方法。容器在询问时查看命名bean的配方，并使用由该bean定义封装的配置元数据来创建（或获取）实际对象。

如果使用基于XML的配置元数据，则指定要在`<bean />`元素的`class`属性中实例化的对象的类型（或类）。这个`class`属性，在内部是一个`BeanDefinition`实例的`Class`属性，通常是强制的。 (对于异常，请参见[“使用实例工厂方法实例化”一节](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/#beans-factory-class-instance-factory-method)和[第3.7节“Bean定义继承”](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/#beans-child-bean-definitions).)使用`Class`属性有两种方法之一：

- 通常，在容器本身通过反射调用其构造函数直接创建bean的情况下指定要构造的bean类，某种程度上等同于使用“new”运算符的Java代码。
- 要指定包含将被调用来创建对象的“static”工厂方法的实际类，类中包含静态方法。从`static`工厂方法的调用返回的对象类型可以是完全相同的类或另一个类。

**内部类名. ** If you want to configure a bean definition for a `static` nested class, you have to use the *binary* name of the nested class.

For example, if you have a class called `Foo` in the `com.example` package, and this `Foo` class has a `static` nested class called `Bar`, the value of the `'class'`attribute on a bean definition would be…​

**内部类名. **如果你想为一个`static'嵌套类配置bean定义，你必须使用嵌套类的* binary *名字。

例如，如果你在`com.example`包中有一个名为`Foo`的类，并且这个`Foo`类有一个叫`Bar`的`static`嵌套类，``class'`attribute的值 一个bean的定义是...​

`com.example.Foo$Bar`

Notice the use of the `$` character in the name to separate the nested class name from the outer class name.
注意在名称中使用`$`字符来分隔嵌套类名和外部类名。

#### Instantiation with a constructor `通过构造函数实例化`

When you create a bean by the constructor approach, all normal classes are usable by and compatible with Spring. That is, the class being developed does not need to implement any specific interfaces or to be coded in a specific fashion. Simply specifying the bean class should suffice. However, depending on what type of IoC you use for that specific bean, you may need a default (empty) constructor.

The Spring IoC container can manage virtually *any* class you want it to manage; it is not limited to managing true JavaBeans. Most Spring users prefer actual JavaBeans with only a default (no-argument) constructor and appropriate setters and getters modeled after the properties in the container. You can also have more exotic non-bean-style classes in your container. If, for example, you need to use a legacy connection pool that absolutely does not adhere to the JavaBean specification, Spring can manage it as well.

With XML-based configuration metadata you can specify your bean class as follows:

当你通过构造函数方法创建一个bean时，所有正常的类都可以被Spring使用并兼容。也就是说，正在开发的类不需要实现任何特定接口或者以特定方式编码。只需指定bean类就足够了。但是，根据您用于该特定bean的IoC的类型，您可能需要一个默认（空）构造函数。

Spring IoC容器可以管理你想要管理的虚拟*任何*类;它不限于管理真正的JavaBean。大多数Spring用户喜欢实际的JavaBeans只有一个默认的（无参数）构造函数和适当的setters和getter在容器中的属性之后建模。你也可以在你的容器中有更多异常的非bean风格的类。例如，如果您需要使用绝对不遵守JavaBean规范的旧连接池，Spring也可以管理它。

使用基于XML的配置元数据，您可以按如下所示指定bean类：


```xml
<bean id="exampleBean" class="examples.ExampleBean"/>

<bean name="anotherExample" class="examples.ExampleBeanTwo"/>
```

有关向构造函数指定参数（如果需要）和在构建对象后设置对象实例属性的机制的详细信息，请参见[依赖注入](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/#beans-factory-collaborators).

#### Instantiation with a static factory method `使用静态工厂方法实例化`

When defining a bean that you create with a static factory method, you use the `class` attribute to specify the class containing the `static` factory method and an attribute named `factory-method` to specify the name of the factory method itself. You should be able to call this method (with optional arguments as described later) and return a live object, which subsequently is treated as if it had been created through a constructor. One use for such a bean definition is to call `static` factories in legacy code.

The following bean definition specifies that the bean will be created by calling a factory-method. The definition does not specify the type (class) of the returned object, only the class containing the factory method. In this example, the `createInstance()` method must be a *static* method.

当定义一个使用静态工厂方法创建的bean时，除了需要指定 class 属性外，还需要通过 factory-method属性来指定创建 bean 实例的工厂方法。Spring将调用此方法(其可选参数接下来介绍)返回实例对象，就此而言，跟通过普通构造器创建类实例没什么两样。

以下bean定义指定将通过调用factory-method创建bean。 该定义不指定返回对象的类型（类），只指定包含工厂方法的类。 在这个例子中，`createInstance()`方法必须是* static *方法。


```xml
<bean id="clientService"
    class="examples.ClientService"
    factory-method="createInstance"/>
```

```java
public class ClientService {
    private static ClientService clientService = new ClientService();
    private ClientService() {}

    public static ClientService createInstance() {
        return clientService;
    }
}
```

有关向工厂方法提供（可选）参数和在工厂返回对象后设置对象实例属性的机制的详细信息，请参阅[依赖和配置详解](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/#beans-factory-properties-detailed).

#### 使用实例工厂方法实例化

Similar to instantiation through a [static factory method](http://docs.spring.io/spring/docs/5.0.0.M3/spring-framework-reference/htmlsingle/#beans-factory-class-static-factory-method), instantiation with an instance factory method invokes a non-static method of an existing bean from the container to create a new bean. To use this mechanism, leave the `class` attribute empty, and in the `factory-bean` attribute, specify the name of a bean in the current (or parent/ancestor) container that contains the instance method that is to be invoked to create the object. Set the name of the factory method itself with the `factory-method` attribute.

类似于通过[静态工厂方法](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/#beans-factory-class-static-factory-method)实例化，使用实例工厂方法的实例化从容器调用现有bean的非静态方法以创建新bean。 要使用此机制，将`class`属性保留为空，并在`factory-bean`属性中，指定当前（或父/祖先）容器中包含要调用的实例方法的bean的名称 创建对象。 使用`factory-method`属性设置工厂方法本身的名称。

```xml
<!-- the factory bean, which contains a method called createInstance() -->
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
    <!-- inject any dependencies required by this locator bean -->
</bean>

<!-- the bean to be created via the factory bean -->
<bean id="clientService"
    factory-bean="serviceLocator"
    factory-method="createClientServiceInstance"/>
```

```java
public class DefaultServiceLocator {

    private static ClientService clientService = new ClientServiceImpl();
    private DefaultServiceLocator() {}

    public ClientService createClientServiceInstance() {
        return clientService;
    }
}
```

One factory class can also hold more than one factory method as shown here:
一个工厂类也可以有多个工厂方法，如下代码所示：

```xml
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
    <!-- inject any dependencies required by this locator bean -->
</bean>

<bean id="clientService"
    factory-bean="serviceLocator"
    factory-method="createClientServiceInstance"/>

<bean id="accountService"
    factory-bean="serviceLocator"
    factory-method="createAccountServiceInstance"/>
```

```java
public class DefaultServiceLocator {

    private static ClientService clientService = new ClientServiceImpl();
    private static AccountService accountService = new AccountServiceImpl();

    private DefaultServiceLocator() {}

    public ClientService createClientServiceInstance() {
        return clientService;
    }

    public AccountService createAccountServiceInstance() {
        return accountService;
    }

}
```

This approach shows that the factory bean itself can be managed and configured through dependency injection (DI). See [Dependencies and configuration in detail](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/#beans-factory-properties-detailed).

| ![[Note]](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/images/note.png) |
| ---------------------------------------- |
|在Spring文档中，* factory bean *是指在Spring容器中配置的bean，它将通过[instance](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/#beans-factory-class-instance-factory-method) 或[static](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/#beans-factory-class-static-factory-method) 工厂方法创建对象。 相比之下，“FactoryBean”（注意大写）指的是Spring特有的[FactoryBean](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/#beans-factory-extension-factorybean). |