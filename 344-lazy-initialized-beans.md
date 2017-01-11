### 3.4.4 Lazy-initialized beans `延迟初始化bean`

By default, `ApplicationContext` implementations eagerly create and configure all [singleton](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/#beans-factory-scopes-singleton) beans as part of the initialization process. Generally, this pre-instantiation is desirable, because errors in the configuration or surrounding environment are discovered immediately, as opposed to hours or even days later. When this behavior is *not* desirable, you can prevent pre-instantiation of a singleton bean by marking the bean definition as lazy-initialized. A lazy-initialized bean tells the IoC container to create a bean instance when it is first requested, rather than at startup.

In XML, this behavior is controlled by the `lazy-init` attribute on the `` element; for example:

默认情况下，`ApplicationContext`实现急切地创建和配置所有[singleton](http://docs.spring.io/spring/docs/5.0.0.M4/spring-framework-reference/htmlsingle/#beans-factory-scopes-singleton) bean作为初始化过程的一部分。 通常，这种预实例化是期望的，因为配置或周围环境中的错误被立即发现，而不是数小时或甚至数天之后。 当这种行为*不是*所期望的，你可以通过将bean定义标记为延迟初始化来阻止单例bean的预实例化。 一个延迟初始化的bean告诉IoC容器在第一次请求时而不是在启动时创建一个bean实例。

在XML中，此行为由`<bean/>`元素上的`lazy-init`属性控制; 例如：

```xml
<bean id="lazy" class="com.foo.ExpensiveToCreateBean" lazy-init="true"/>
<bean name="not.lazy" class="com.foo.AnotherBean"/>
```

When the preceding configuration is consumed by an `ApplicationContext`, the bean named `lazy` is not eagerly pre-instantiated when the `ApplicationContext` is starting up, whereas the `not.lazy` bean is eagerly pre-instantiated.

However, when a lazy-initialized bean is a dependency of a singleton bean that is *not* lazy-initialized, the `ApplicationContext` creates the lazy-initialized bean at startup, because it must satisfy the singleton’s dependencies. The lazy-initialized bean is injected into a singleton bean elsewhere that is not lazy-initialized.

You can also control lazy-initialization at the container level by using the `default-lazy-init` attribute on the `<beans/>` element; for example:

当前面的配置被一个`ApplicationContext`消费时，名为`lazy`的bean在ApplicationContext启动时不会被预先实例化，而`not.lazy`bean被预先实例化。

但是，当一个延迟初始化的bean是单例bean的依赖，而这个单例bean又*不是* lazy初始化时，ApplicationContext在启动时创建延迟初始化的bean，因为它必须满足singleton的依赖。因此延迟加载的bean会被注入单例bean。

您还可以通过使用`<bean />`元素上的`default-lazy-init`属性在容器级别控制延迟初始化; 例如：

```xml
<beans default-lazy-init="true">
	<!-- no beans will be pre-instantiated... -->
</beans>
```