### 3.4.1 Dependency Injection `依赖注入`

*Dependency injection* (DI) is a process whereby objects define their dependencies, that is, the other objects they work with, only through constructor arguments, arguments to a factory method, or properties that are set on the object instance after it is constructed or returned from a factory method. The container then *injects* those dependencies when it creates the bean. This process is fundamentally the inverse, hence the name *Inversion of Control* (IoC), of the bean itself controlling the instantiation or location of its dependencies on its own by using direct construction of classes, or the *Service Locator* pattern.

Code is cleaner with the DI principle and decoupling is more effective when objects are provided with their dependencies. The object does not look up its dependencies, and does not know the location or class of the dependencies. As such, your classes become easier to test, in particular when the dependencies are on interfaces or abstract base classes, which allow for stub or mock implementations to be used in unit tests.

DI exists in two major variants, [Constructor-based dependency injection](http://docs.spring.io/spring/docs/5.0.0.M3/spring-framework-reference/htmlsingle/#beans-constructor-injection) and [Setter-based dependency injection](http://docs.spring.io/spring/docs/5.0.0.M3/spring-framework-reference/htmlsingle/#beans-setter-injection).

*依赖注入*（DI）是一个过程，对象通过构造函数参数，工厂方法的参数或在构造对象实例后设置的属性来定义它们的依赖关系，即它们所处理的其他对象或从工厂方法返回。容器然后*在创建bean时注入*这些依赖。这个过程基本上是相反的，因此名称为* Inversion of Control *（IoC），通过使用类的直接构造或* Service Locator *模式来控制其自身的依赖性来实例化或通过调用的bean自身的名称(比如xml里的那些配置，通过名字直接调用到)。

代码与DI原理更清洁，当对象提供其依赖性时，解耦更有效。该对象不查找其依赖关系，并且不知道依赖关系的位置或类。因此，您的类变得更容易测试，特别是当依赖关系在接口或抽象基类上时，这允许在单元测试中使用存根或模拟实现。

DI存在两个主要形式，[基于构造函数的依赖注入](http://docs.spring.io/spring/docs/5.0.0.M3/spring-framework-reference/htmlsingle/#beans-constructor-injection) 和[基于Setter的依赖注入](http://docs.spring.io/spring/docs/5.0.0.M3/spring-framework-reference/htmlsingle/#beans-setter-injection)。

#### 基于构造方法的依赖注入

*Constructor-based* DI is accomplished by the container invoking a constructor with a number of arguments, each representing a dependency. Calling a `static` factory method with specific arguments to construct the bean is nearly equivalent, and this discussion treats arguments to a constructor and to a `static` factory method similarly. The following example shows a class that can only be dependency-injected with constructor injection. Notice that there is nothing *special* about this class, it is a POJO that has no dependencies on container specific interfaces, base classes or annotations.

*基于构造函数的* DI由容器调用具有多个参数的构造函数完成，每个参数表示依赖。 调用具有特定参数的`static`工厂方法来构造bean几乎是等效的，并且这个结论同样对将参数传递给构造函数和静态工厂方法有效。 以下示例显示只能使用构造函数注入进行依赖关系注入的类。 注意，这个类没有什么特别的*，它是一个POJO没有依赖于容器特定的接口，基类或注释。

```java
public class SimpleMovieLister {

    // the SimpleMovieLister has a dependency on a MovieFinder
    private MovieFinder movieFinder;

    // a constructor so that the Spring container can inject a MovieFinder
    public SimpleMovieLister(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // business logic that actually uses the injected MovieFinder is omitted...

}
```

#### 基于Setter的依赖注入

*Setter-based* DI is accomplished by the container calling setter methods on your beans after invoking a no-argument constructor or no-argument `static` factory method to instantiate your bean.

The following example shows a class that can only be dependency-injected using pure setter injection. This class is conventional Java. It is a POJO that has no dependencies on container specific interfaces, base classes or annotations.

*基于Setter的* DI是通过在调用一个无参构造函数或无参数“static”工厂方法来实例化你的bean之后，通过容器调用你的bean上的setter方法来完成的。

以下示例显示只能使用纯setter注入进行依赖关系注入的类。 这个类是常规的Java。 它是一个POJO，没有依赖于容器特定的接口，基类或注释。


```java
public class SimpleMovieLister {

    // the SimpleMovieLister has a dependency on the MovieFinder
    private MovieFinder movieFinder;

    // a setter method so that the Spring container can inject a MovieFinder
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // business logic that actually uses the injected MovieFinder is omitted...

}
```

The `ApplicationContext` supports constructor-based and setter-based DI for the beans it manages. It also supports setter-based DI after some dependencies have already been injected through the constructor approach. You configure the dependencies in the form of a `BeanDefinition`, which you use in conjunction with `PropertyEditor` instances to convert properties from one format to another. However, most Spring users do not work with these classes directly (i.e., programmatically) but rather with XML `bean` definitions, annotated components (i.e., classes annotated with `@Component`, `@Controller`, etc.), or `@Bean` methods in Java-based `@Configuration` classes. These sources are then converted internally into instances of `BeanDefinition` and used to load an entire Spring IoC container instance.

`ApplicationContext`支持其管理的bean的基于构造函数和基于setter的DI。 它还支持基于setter的DI，在一些依赖关系已经通过构造函数方法注入之后。 您以一个`BeanDefinition`的形式配置依赖关系，你可以与PropertyEditor实例结合使用将属性从一种格式转换为另一种格式。 然而，大多数Spring用户不直接使用这些类（即，以编程方式），而是使用XML`bean`定义，或注解组件（即用`@ Component`，`@ Controller`等注解的类） @ Bean`方法在基于Java的`@ Configuration`类中定义。 然后通过这些配置在内部转换为`BeanDefinition`的实例，并用于加载整个Spring IoC容器实例。

**基于构造函数或基于setter的DI？**

Since you can mix constructor-based and setter-based DI, it is a good rule of thumb to use constructors for *mandatory dependencies* and setter methods or configuration methods for *optional dependencies*. Note that use of the [@Required](http://docs.spring.io/spring/docs/5.0.0.M3/spring-framework-reference/htmlsingle/#beans-required-annotation) annotation on a setter method can be used to make the property a required dependency.

The Spring team generally advocates constructor injection as it enables one to implement application components as *immutable objects* and to ensure that required dependencies are not `null`. Furthermore constructor-injected components are always returned to client (calling) code in a fully initialized state. As a side note, a large number of constructor arguments is a *bad code smell*, implying that the class likely has too many responsibilities and should be refactored to better address proper separation of concerns.

Setter injection should primarily only be used for optional dependencies that can be assigned reasonable default values within the class. Otherwise, not-null checks must be performed everywhere the code uses the dependency. One benefit of setter injection is that setter methods make objects of that class amenable to reconfiguration or re-injection later. Management through [JMX MBeans](http://docs.spring.io/spring/docs/5.0.0.M3/spring-framework-reference/htmlsingle/#jmx) is therefore a compelling use case for setter injection.

Use the DI style that makes the most sense for a particular class. Sometimes, when dealing with third-party classes for which you do not have the source, the choice is made for you. For example, if a third-party class does not expose any setter methods, then constructor injection may be the only available form of DI.

因为你可以混合基于构造函数和基于setter的DI，使用构造函数*强制依赖 *和setter方法或*可选依赖*的配置方法是一个好的经验法则。请注意，在设置方法上使用 [@Required](http://docs.spring.io/spring/docs/5.0.0.M3/spring-framework-reference/htmlsingle/#beans-required-annotation) 注解可以用于使属性成为必需的依赖关系。

Spring团队通常倡导构造函数注入，因为它能够将应用程序组件实现为* immutable对象*，并确保所需的依赖项不是“null”。此外，构造器注入的组件总是返回到完全初始化状态下的客户端（调用）代码。需要多说一点，大量的构造函数参数是一个*坏的代码气味*，暗示该类可能有太多的责任，应该重构，以更好地解耦。

Setter注入应主要仅用于可以在类中分配合理默认值的可选依赖项。否则，必须在代码使用依赖性的任何地方执行非空检查。 setter注入的一个好处是setter方法使该类的对象可以重新配置或稍后重新注入。因此，通过[JMX MBeans](http://docs.spring.io/spring/docs/5.0.0.M3/spring-framework-reference/htmlsingle/#jmx)的管理是setter注入的一个让人眼前一亮的例子。

使用对特定类最有意义的DI样式。有时候，当你处理你没有源码的第三方类时，选择你所能做到的。例如，如果第三方类没有公开任何setter方法，则构造函数注入可能是DI的唯一可用形式。

####依赖解析过程

The container performs bean dependency resolution as follows:

- The `ApplicationContext` is created and initialized with configuration metadata that describes all the beans. Configuration metadata can be specified via XML, Java code, or annotations.
- For each bean, its dependencies are expressed in the form of properties, constructor arguments, or arguments to the static-factory method if you are using that instead of a normal constructor. These dependencies are provided to the bean, *when the bean is actually created*.
- Each property or constructor argument is an actual definition of the value to set, or a reference to another bean in the container.
- Each property or constructor argument which is a value is converted from its specified format to the actual type of that property or constructor argument. By default Spring can convert a value supplied in string format to all built-in types, such as `int`, `long`, `String`, `boolean`, etc.

The Spring container validates the configuration of each bean as the container is created. However, the bean properties themselves are not set until the bean *is actually created*. Beans that are singleton-scoped and set to be pre-instantiated (the default) are created when the container is created. Scopes are defined in [Section 3.5, “Bean scopes”](http://docs.spring.io/spring/docs/5.0.0.M3/spring-framework-reference/htmlsingle/#beans-factory-scopes). Otherwise, the bean is created only when it is requested. Creation of a bean potentially causes a graph of beans to be created, as the bean’s dependencies and its dependencies' dependencies (and so on) are created and assigned. Note that resolution mismatches among those dependencies may show up late, i.e. on first creation of the affected bean.

容器对bean依赖性解析过程如下：

- 使用描述所有bean的配置元数据创建和初始化ApplicationContext。配置元数据可以通过XML，Java代码或注解指定。
- 对于每个bean，它的依赖关系以属性，构造函数参数或静态工厂方法的参数的形式表示，如果你使用而不是一个正常的构造函数。这些依赖关系提供给bean，实际创建bean时调用。
- 每个属性或构造函数参数是要设置的值的实际定义，或对容器中另一个bean的引用。
- 作为值的每个属性或构造函数参数从其指定的格式转换为该属性或构造函数参数的实际类型。默认情况下，Spring可以将以字符串格式提供的值转换为所有内置类型，例如`int`，`long`，`String`，`boolean`等。

Spring容器在创建容器时验证每个bean的配置。但是，bean属性本身不会设置，直到实际创建的时候调用。在创建容器时创建单例范围并设置为预实例化的Bean（默认值）。范围在[第3.5节 “Bean scopes”](http://docs.spring.io/spring/docs/5.0.0.M3/spring-framework-reference/htmlsingle/#beans-factory-scopes)中定义。否则，仅当请求时才创建bean。创建bean可能导致创建bean的图形，因为bean的依赖关系及其依赖关系（依此类推）被创建和分配。注意，那些依赖关系之间的分辨率不匹配可能迟到，即首次创建受影响的bean时显示。

**循环依赖**

If you use predominantly constructor injection, it is possible to create an unresolvable circular dependency scenario.

For example: Class A requires an instance of class B through constructor injection, and class B requires an instance of class A through constructor injection. If you configure beans for classes A and B to be injected into each other, the Spring IoC container detects this circular reference at runtime, and throws a`BeanCurrentlyInCreationException`.

One possible solution is to edit the source code of some classes to be configured by setters rather than constructors. Alternatively, avoid constructor injection and use setter injection only. In other words, although it is not recommended, you can configure circular dependencies with setter injection.

Unlike the *typical* case (with no circular dependencies), a circular dependency between bean A and bean B forces one of the beans to be injected into the other prior to being fully initialized itself (a classic chicken/egg scenario).

You can generally trust Spring to do the right thing. It detects configuration problems, such as references to non-existent beans and circular dependencies, at container load-time. Spring sets properties and resolves dependencies as late as possible, when the bean is actually created. This means that a Spring container which has loaded correctly can later generate an exception when you request an object if there is a problem creating that object or one of its dependencies. For example, the bean throws an exception as a result of a missing or invalid property. This potentially delayed visibility of some configuration issues is why `ApplicationContext` implementations by default pre-instantiate singleton beans. At the cost of some upfront time and memory to create these beans before they are actually needed, you discover configuration issues when the `ApplicationContext` is created, not later. You can still override this default behavior so that singleton beans will lazy-initialize, rather than be pre-instantiated.

If no circular dependencies exist, when one or more collaborating beans are being injected into a dependent bean, each collaborating bean is *totally* configured prior to being injected into the dependent bean. This means that if bean A has a dependency on bean B, the Spring IoC container completely configures bean B prior to invoking the setter method on bean A. In other words, the bean is instantiated (if not a pre-instantiated singleton), its dependencies are set, and the relevant lifecycle methods (such as a [configured init method](http://docs.spring.io/spring/docs/5.0.0.M3/spring-framework-reference/htmlsingle/#beans-factory-lifecycle-initializingbean) or the [InitializingBean callback method](http://docs.spring.io/spring/docs/5.0.0.M3/spring-framework-reference/htmlsingle/#beans-factory-lifecycle-initializingbean)) are invoked.



如果主要使用构造函数注入，可以创建一个不可解析的循环依赖场景。

例如：A类通过构造函数注入需要B类的实例，B类通过构造函数注入需要A类的实例。如果将A和B类的bean配置为彼此注入，则Spring IoC容器在运行时检测到此循环引用，并抛出一个`BeanCurrentlyInCreationException`。

一个可行的解决方案是编辑要由setter而不是构造函数配置的一些类的源代码。或者，避免构造函数注入，并仅使用setter注入。换句话说，虽然不推荐，可以使用setter注入配置循环依赖性。

与典型情况（没有循环依赖）不同，bean A和bean B之间的循环依赖性迫使一个bean在被完全初始化之前被注入另一个bean（一个经典的鸡/鸡蛋场景）。

你一般可以信任Spring做正确的事情。它在容器加载时检测配置问题，例如引用不存在的bean和循环依赖性。 Spring在实际创建bean时尽可能晚地设置属性并解析依赖关系。这意味着，如果在创建该对象或其某个依赖关系时出现问题，则在请求对象时，已正确加载的Spring容器可能稍后会生成异常。例如，bean由于缺少或无效的属性而抛出异常。这可能延迟一些配置问题的可见性是因为ApplicationContext实现这默认情况下预实例化单例bean。以实际需要之前创建这些bean的一些预先时间和内存为代价，您在创建ApplicationContext时不会发现配置问题。您仍然可以覆盖此默认行为，以便单例bean将延迟初始化，而不是预先实例化(其实就是懒加载，真正用到的时候才会发生，提高了系统的性能)。

如果不存在循环依赖性，则当一个或多个协作bean被注入到依赖bean中时，每个协作bean在被注入到依赖bean之前被完全配置。这意味着如果bean A对bean B有依赖性，则Spring IoC容器在调用bean A上的setter方法之前完全配置bean B.换句话说，bean被实例化（如果不是实例化的单例）设置相关性，并调用相关的生命周期方法（如配置的init方法或InitializingBean回调方法）。

#### 依赖注入示例

The following example uses XML-based configuration metadata for setter-based DI. A small part of a Spring XML configuration file specifies some bean definitions:
以下示例使用基于XML的配置基于setter的DI。 Spring XML配置文件的一小部分指定了一些bean定义：

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <!-- setter injection using the nested ref element -->
    <property name="beanOne">
        <ref bean="anotherExampleBean"/>
    </property>

    <!-- setter injection using the neater ref attribute -->
    <property name="beanTwo" ref="yetAnotherBean"/>
    <property name="integerProperty" value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```

```java
public class ExampleBean {

    private AnotherBean beanOne;
    private YetAnotherBean beanTwo;
    private int i;

    public void setBeanOne(AnotherBean beanOne) {
        this.beanOne = beanOne;
    }

    public void setBeanTwo(YetAnotherBean beanTwo) {
        this.beanTwo = beanTwo;
    }

    public void setIntegerProperty(int i) {
        this.i = i;
    }

}
```

In the preceding example, setters are declared to match against the properties specified in the XML file. The following example uses constructor-based DI:

在前面的示例中，setters被声明为与XML文件中指定的属性匹配。 以下示例使用基于构造函数的DI：

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <!-- constructor injection using the nested ref element -->
    <constructor-arg>
        <ref bean="anotherExampleBean"/>
    </constructor-arg>

    <!-- constructor injection using the neater ref attribute -->
    <constructor-arg ref="yetAnotherBean"/>

    <constructor-arg type="int" value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```

```java
public class ExampleBean {

    private AnotherBean beanOne;
    private YetAnotherBean beanTwo;
    private int i;

    public ExampleBean(
        AnotherBean anotherBean, YetAnotherBean yetAnotherBean, int i) {
        this.beanOne = anotherBean;
        this.beanTwo = yetAnotherBean;
        this.i = i;
    }

}
```

The constructor arguments specified in the bean definition will be used as arguments to the constructor of the `ExampleBean`.

Now consider a variant of this example, where instead of using a constructor, Spring is told to call a `static` factory method to return an instance of the object:

在bean定义中指定的构造函数参数将被用作`ExampleBean`构造函数的参数。

现在考虑这个例子的一个变体，在这里不使用构造函数，Spring被告知要调用一个`static`工厂方法来返回一个对象的实例：

```java
<bean id="exampleBean" class="examples.ExampleBean" factory-method="createInstance">
    <constructor-arg ref="anotherExampleBean"/>
    <constructor-arg ref="yetAnotherBean"/>
    <constructor-arg value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```

```java
public class ExampleBean {

    // a private constructor
    private ExampleBean(...) {
        ...
    }

    // a static factory method; the arguments to this method can be
    // considered the dependencies of the bean that is returned,
    // regardless of how those arguments are actually used.
    public static ExampleBean createInstance (
        AnotherBean anotherBean, YetAnotherBean yetAnotherBean, int i) {

        ExampleBean eb = new ExampleBean (...);
        // some other operations...
        return eb;
    }

}
```

Arguments to the `static` factory method are supplied via undefined`undefined` elements, exactly the same as if a constructor had actually been used. The type of the class being returned by the factory method does not have to be of the same type as the class that contains the `static` factory method, although in this example it is. An instance (non-static) factory method would be used in an essentially identical fashion (aside from the use of the `factory-bean` attribute instead of the `class` attribute), so details will not be discussed here.

`static`工厂方法的参数是通过未定义的`<constructor-arg/>`元素提供的，完全和实际使用的构造函数一样。 由工厂方法返回的类的类型不必与包含“static”工厂方法的类的类型相同，尽管在本例中它是。 一个实例（非静态）工厂方法将以基本相同的方式使用（除了使用“factory-bean”属性而不是“class”属性），因此这里不再讨论细节。



