### 3.4.1 Dependency Injection `依赖注入`

*Dependency injection* (DI) is a process whereby objects define their dependencies, that is, the other objects they work with, only through constructor arguments, arguments to a factory method, or properties that are set on the object instance after it is constructed or returned from a factory method. The container then *injects* those dependencies when it creates the bean. This process is fundamentally the inverse, hence the name *Inversion of Control* (IoC), of the bean itself controlling the instantiation or location of its dependencies on its own by using direct construction of classes, or the *Service Locator* pattern.

Code is cleaner with the DI principle and decoupling is more effective when objects are provided with their dependencies. The object does not look up its dependencies, and does not know the location or class of the dependencies. As such, your classes become easier to test, in particular when the dependencies are on interfaces or abstract base classes, which allow for stub or mock implementations to be used in unit tests.

DI exists in two major variants, [Constructor-based dependency injection](http://docs.spring.io/spring/docs/5.0.0.M3/spring-framework-reference/htmlsingle/#beans-constructor-injection) and [Setter-based dependency injection](http://docs.spring.io/spring/docs/5.0.0.M3/spring-framework-reference/htmlsingle/#beans-setter-injection).

*依赖注入*（DI）是一个过程，对象通过构造函数参数，工厂方法的参数或在构造对象实例后设置的属性来定义它们的依赖关系，即它们所处理的其他对象或从工厂方法返回。容器然后*在创建bean时注入*这些依赖。这个过程基本上是相反的，因此名称为* Inversion of Control *（IoC），通过使用类的直接构造或* Service Locator *模式来控制其自身的依赖性来实例化或通过调用的bean自身的名称(比如xml里的那些配置，通过名字直接调用到)。

代码与DI原理更清洁，当对象提供其依赖性时，解耦更有效。该对象不查找其依赖关系，并且不知道依赖关系的位置或类。因此，您的类变得更容易测试，特别是当依赖关系在接口或抽象基类上时，这允许在单元测试中使用存根或模拟实现。

DI存在两个主要形式，[基于构造函数的依赖注入](http://docs.spring.io/spring/docs/5.0.0.M3/spring-framework-reference/htmlsingle/#beans-constructor-injection) 和[基于Setter的依赖注入](http://docs.spring.io/spring/docs/5.0.0.M3/spring-framework-reference/htmlsingle/#beans-setter-injection)。
