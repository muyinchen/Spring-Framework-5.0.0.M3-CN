In most application scenarios, most beans in the container are [singletons](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/#beans-factory-scopes-singleton). When a singleton bean needs to collaborate with another singleton bean, or a non-singleton bean needs to collaborate with another non-singleton bean, you typically handle the dependency by defining one bean as a property of the other. A problem arises when the bean lifecycles are different. Suppose singleton bean A needs to use non-singleton (prototype) bean B, perhaps on each method invocation on A. The container only creates the singleton bean A once, and thus only gets one opportunity to set the properties. The container cannot provide bean A with a new instance of bean B every time one is needed.

A solution is to forego some inversion of control. You can [make bean A aware of the container](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/#beans-factory-aware) by implementing the `ApplicationContextAware` interface, and by [making a getBean("B") call to the container](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/#beans-factory-client) ask for (a typically new) bean B instance every time bean A needs it. The following is an example of this approach:

在大多数应用场景中，容器中的大多数bean都是 [singletons](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/#beans-factory-scopes-singleton)。 当单例bean需要与另一个单例bean协作或非单例bean需要与另一个非单例bean协作时，通常通过将一个bean定义为另一个的属性来处理依赖关系。不过对于具有不同生命周期的bean 来说这样做就会出现问题。 假设单例bean A需要使用非单例（原型）bean B，也许在A上的每个方法调用上。容器仅创建单例bean A一次，因此只有一次机会来设置属性。 这样就没办法 在需要的时候每次让容器为bean A提供一个新的bean B实例。

解决方案是放弃一些控制的反转。 您可以通过实现以下操作来[让`bean A`知道容器](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/#beans-factory-aware)  `ApplicationContextAware`接口，并通过[对容器调用getBean（“B”）](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/#beans-factory-client)在每次bean A需要它时请求（一个通常是新的）bean B实例。 以下是此方法的示例：
```java
// a class that uses a stateful Command-style class to perform some processing
package fiona.apple;

// Spring-API imports
import org.springframework.beans.BeansException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;

public class CommandManager implements ApplicationContextAware {

	private ApplicationContext applicationContext;

	public Object process(Map commandState) {
		// grab a new instance of the appropriate Command
		Command command = createCommand();
		// set the state on the (hopefully brand new) Command instance
		command.setState(commandState);
		return command.execute();
	}

	protected Command createCommand() {
		// notice the Spring API dependency!
		return this.applicationContext.getBean("command", Command.class);
	}

	public void setApplicationContext(
			ApplicationContext applicationContext) throws BeansException {
		this.applicationContext = applicationContext;
	}
}
```

The preceding is not desirable, because the business code is aware of and coupled to the Spring Framework. Method Injection, a somewhat advanced feature of the Spring IoC container, allows this use case to be handled in a clean fashion.

You can read more about the motivation for Method Injection in [this blog entry](https://spring.io/blog/2004/08/06/method-injection/).

前面是不可取的，，因为业务代码和Spring框架产生的耦合。方法注入，作为Spring Ioc容器的高级特性，，允许以干净的方式处理这个用例。

你可以在[这个博客查看](https://spring.io/blog/2004/08/06/method-injection/)阅读更多关于方法注入的动机

#### Lookup method injection

查找方法注入具有使容器覆盖*受容器管理的bean *上的方法的能力，从而可以返回容器中另一个命名bean的查找结果。 查找方法注入适用于原型bean，如前一节中所述的场景。 Spring框架通过使用从CGLIB库生成的字节码来动态生成覆盖该方法的子类来实现此方法注入。

| ![[Note]](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| ---------------------------------------- |
| 为了使这个动态子类化起作用，Spring bean容器将继承的类不能是`final`，被重写的方法也不能是`final`。对一个具有`abstract`方法的类进行单元测试需要你为其写一个子类并提供该`抽象方法`的实现。Concrete(具体)方法对于组件扫描也是必要的，这需要具体的类来拾取。另一个关键的限制是，查找方法将不能使用工厂方法, 特别是在配置类中不使用`@Bean`方法，因为容器不负责在这种情况下创建实例，因此不能即时创建运行时生成的子类。 |

看看前面代码片段中的`CommandManager`类，你会看到Spring容器将动态覆盖`createCommand()`方法的实现。 你的`CommandManager`类不会有任何Spring依赖，在下面重写的例子中可以看出：
```java
package fiona.apple;

// no more Spring imports!

public abstract class CommandManager {

	public Object process(Object commandState) {
		// grab a new instance of the appropriate Command interface
		Command command = createCommand();
		// set the state on the (hopefully brand new) Command instance
		command.setState(commandState);
		return command.execute();
	}

	// okay... but where is the implementation of this method?
	protected abstract Command createCommand();
}
```

在包含要注入的方法（在这种情况下为“CommandManager”）的客户端类中，要注入的方法需要具有以下形式的签名：
```java
<public|protected> [abstract] <return-type> theMethodName(no-arguments);
```

如果方法是“抽象的”，动态生成的子类实现该方法。 否则，动态生成的子类将覆盖原始类中定义的具体方法。 例如：

```java
<!-- a stateful bean deployed as a prototype (non-singleton) -->
<bean id="myCommand" class="fiona.apple.AsyncCommand" scope="prototype">
	<!-- inject dependencies here as required -->
</bean>

<!-- commandProcessor uses statefulCommandHelper -->
<bean id="commandManager" class="fiona.apple.CommandManager">
	<lookup-method name="createCommand" bean="myCommand"/>
</bean>
```

标识为* commandManager *的bean在需要`myCommand`  bean的新实例时，将调用自己的`createCommand()`方法。 将`myCommand` bean 设置成prototype，如果这是实际上需要的话,一定要谨慎处理。 如果它是一个 [singleton](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/#beans-factory-scopes-singleton)，那么每次将返回相同的`myCommand` bean。

或者，在基于注解的组件模型中，您可以通过“@Lookup”注释声明一个查找方法：

```java
public abstract class CommandManager {

	public Object process(Object commandState) {
		Command command = createCommand();
		command.setState(commandState);
		return command.execute();
	}

	@Lookup("myCommand")
	protected abstract Command createCommand();
}
```


或者，更常用点，你可以依赖于目标bean根据查找方法的声明返回类型得到解决：

```java
public abstract class CommandManager {

	public Object process(Object commandState) {
		MyCommand command = createCommand();
		command.setState(commandState);
		return command.execute();
	}

	@Lookup
	protected abstract MyCommand createCommand();
}
```

注意，您通常会使用具体的子类实现来声明这种带注解的查找方法，以使它们与Spring的组件扫描规则兼容，默认情况下会忽略抽象类。 此限制不适用于显式注册或显式导入的bean类的情况。

| ![[Tip]](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/images/tip.png.pagespeed.ce.w22Wv-tZ37.png) |
| ---------------------------------------- |
| 另一种访问不同范围的目标bean的方法是`ObjectFactory`/ `Provider`注入点。 查看[the section called “Scoped beans as dependencies”](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/#beans-factory-scopes-other-injection)。感兴趣的读者也可以找到`ServiceLocatorFactoryBean`（在`org.springframework.beans.factory.config`包中）来用一用。 |

#### 任意方法替换


与查找方法注入相比，很少有用的方法注入形式是使用另一个方法实现替换托管bean中的任意方法的能力。 用户可以安全地跳过本节的其余部分，直到实际需要该功能。

使用基于XML的配置元数据，您可以使用“replaced-method”元素将现有的方法实现替换为另一个已部署的bean。 考虑下面的类，使用方法computeValue，我们要覆盖：

```java
public class MyValueCalculator {

	public String computeValue(String input) {
		// some real code...
	}

	// some other methods...

}
```

实现`org.springframework.beans.factory.support.MethodReplacer`接口的类提供了新的方法定义。

```java
/**
 * meant to be used to override the existing computeValue(String)
 * implementation in MyValueCalculator
 */
public class ReplacementComputeValue implements MethodReplacer {

	public Object reimplement(Object o, Method m, Object[] args) throws Throwable {
		// get the input value, work with it, and return a computed result
		String input = (String) args[0];
		...
		return ...;
	}
}
```

下面的bean定义中指定了要配置的原始类和将要重写的方法：

```java
<bean id="myValueCalculator" class="x.y.z.MyValueCalculator">
	<!-- arbitrary method replacement -->
	<replaced-method name="computeValue" replacer="replacementComputeValue">
		<arg-type>String</arg-type>
	</replaced-method>
</bean>

<bean id="replacementComputeValue" class="a.b.c.ReplacementComputeValue"/>
```

您可以在`<replaced-method/>`元素中使用一个或多个包含的`<arg-type/>`元素来指示要覆盖的方法的方法签名。 仅当方法重载并且类中存在多个变量时，参数的签名才是必需的。 为了方便起见，参数的类型字符串可以是完全限定类型名称的子字符串。 例如，以下所有匹配`java.lang.String`：

```java
java.lang.String
String
Str
```

因为参数的数量通常足以区分每个可能的选择，这个简写方式可以节省大量的输入，允许你只需要输入最短字符串就可匹配参数类型。