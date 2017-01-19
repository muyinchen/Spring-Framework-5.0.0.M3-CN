In most application scenarios, most beans in the container are [singletons](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/#beans-factory-scopes-singleton). When a singleton bean needs to collaborate with another singleton bean, or a non-singleton bean needs to collaborate with another non-singleton bean, you typically handle the dependency by defining one bean as a property of the other. A problem arises when the bean lifecycles are different. Suppose singleton bean A needs to use non-singleton (prototype) bean B, perhaps on each method invocation on A. The container only creates the singleton bean A once, and thus only gets one opportunity to set the properties. The container cannot provide bean A with a new instance of bean B every time one is needed.

A solution is to forego some inversion of control. You can [make bean A aware of the container](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/#beans-factory-aware) by implementing the `ApplicationContextAware` interface, and by [making a getBean("B") call to the container](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/#beans-factory-client) ask for (a typically new) bean B instance every time bean A needs it. The following is an example of this approach:

在大多数应用场景中，容器中的大多数bean都是 [singletons](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/#beans-factory-scopes-singleton)。 当单例bean需要与另一个单例bean协作或非单例bean需要与另一个非单例bean协作时，通常通过将一个bean定义为另一个的属性来处理依赖关系。不过对于具有不同生命周期的bean 来说这样做就会出现问题。 假设单例bean A需要使用非单例（原型）bean B，也许在A上的每个方法调用上。容器仅创建单例bean A一次，因此只有一次机会来设置属性。 这样就没办法 在需要的时候每次让容器为bean A提供一个新的bean B实例。

解决方案是放弃一些控制的反转。 您可以通过实现以下操作来[让bean A知道容器](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/#beans-factory-aware)  `ApplicationContextAware`接口，并通过[对容器调用getBean（“B”）](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/#beans-factory-client)在每次bean A需要它时请求（一个通常是新的）bean B实例。 以下是此方法的示例：
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