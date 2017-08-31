### 16.4.1Spring中JPA配置的三个选项

Spring JPA支持提供了三种配置JPA`EntityManagerFactory`的方法，之后通过`EntityManagerFactory`来获取对应的实体管理器。

#### LocalEntityManagerFactoryBean

| ![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| :--- |
| 通常只有在简单的部署环境中使用此选项，例如在独立应用程序或者进行集成测试时，才会使用这种方式。 |

`LocalEntityManagerFactoryBean`创建一个适用于应用程序且仅使用JPA进行数据访问的简单部署环境的`EntityManagerFactory`。工厂bean会使用JPA`PersistenceProvider`自动检测机制，并且在大多数情况下，仅要求开发者指定持久化单元的名称：

```java
<beans>
    <bean id="myEmf" class="org.springframework.orm.jpa.LocalEntityManagerFactoryBean">
        <property name="persistenceUnitName" value="myPersistenceUnit"/>
    </bean>
</beans>
```

这种形式的JPA部署是最简单的，同时限制也很多。开发者不能引用现有的JDBC `DataSource`bean定义，并且不支持全局事务。而且，持久化类的织入\(weaving\)\(字节码转换\)是特定于提供者的，通常需要在启动时指定特定的JVM代理。该选项仅适用于符合JPA Spec的独立应用程序或测试环境。

#### 从JNDI中获取EntityManagerFactory {#从jndi中获取entitymanagerfactory}

| ![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| :--- |
| 在部署到J2EE服务器时可以使用此选项。检查服务器的文档来了解如何将自定义JPA提供程序部署到服务器中，从而对服务器进行比默认更多的个性化定制。 |

从JNDI获取`EntityManagerFactory`\(例如在Java EE环境中\)，只需要在XML配置中加入配置信息即可：

```java
<beans>
    <jee:jndi-lookup id="myEmf" jndi-name="persistence/myPersistenceUnit"/>
</beans>
```

此操作将采用标准J2EE引导：J2EE服务器自动检测J2EE部署描述符（例如web.xml）中`persistence-unit-ref`条目和持久性单元（实际上是应用程序jar中的`META-INF/persistence.xml`文件），并为这些持久性单元定义环境上下文位置。

在这种情况下，整个持久化单元部署（包括持久化类的织入\(weaving\)（字节码转换））都取决于J2EE服务器。JDBC`DataSource`通过`META-INF/persistence.xml`文件中的JNDI位置进行定义; 而`EntityManager`事务与服务器JTA子系统集成。 Spring仅使用获取的`EntityManagerFactory`，通过依赖注入将其传递给应用程序对象，通常通过`JtaTransactionManager`来管理持久性单元的事务。

如果在同一应用程序中使用多个持久性单元，则这种JNDI检索的持久性单元的bean名称应与应用程序用于引用它们的持久性单元名称相匹配，例如`@PersistenceUnit`和`@PersistenceContext`注释。

#### LocalContainerEntityManagerFactoryBean

| ![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| :--- |
| 在基于Spring的应用程序环境中使用此选项来实现完整的JPA功能。这包括诸如Tomcat的Web容器，以及具有复杂持久性要求的独立应用程序和集成测试。 |

`LocalContainerEntityManagerFactoryBean`可以完全控制`EntityManagerFactory`的配置，同时适用于需要细粒度定制的环境。

`LocalContainerEntityManagerFactoryBean`会基于`persistence.xml`文件，`dataSourceLookup`策略和指定的`loadTimeWeaver`来创建一个`PersistenceUnitInfo`实例。因此，可以在JNDI之外使用自定义数据源并控制织入\(weaving\)过程。以下示例显示`LocalContainerEntityManagerFactoryBean`的典型Bean定义：

```java
<beans>
    <bean id="myEmf" class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
        <property name="dataSource" ref="someDataSource"/>
        <property name="loadTimeWeaver">
            <bean class="org.springframework.instrument.classloading.InstrumentationLoadTimeWeaver"/>
        </property>
    </bean>
</beans>
```

下面的例子是一个典型的`persistence.xml`文件：

```java
<persistence xmlns="http://java.sun.com/xml/ns/persistence" version="1.0">
    <persistence-unit name="myUnit" transaction-type="RESOURCE_LOCAL">
        <mapping-file>META-INF/orm.xml</mapping-file>
        <exclude-unlisted-classes/>
    </persistence-unit>
</persistence>
```

| ![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| :--- |
| `<exclude-unlisted-classes />`标签表示不会进行注解实体类的扫描。指定的显式`true`值 – `<exclude-unlisted-classes>true</exclude-unlisted-classes/>`– 也意味着不进行扫描。`<exclude-unlisted-classes> false</exclude-unlisted-classes>`则会触发扫描;但是，如果开发者需要进行实体类扫描，建议开发者简单地省略`<exclude-unlisted-classes>`元素。 |

`LocalContainerEntityManagerFactoryBean`是最强大的JPA设置选项，允许在应用程序中进行灵活的本地配置。它支持连接到现有的JDBC`DataSource`，支持本地和全局事务等。但是，它对运行时环境施加了需求，其中之一就是如果持久性提供程序需要字节码转换，就需要有织入\(weaving\)能力的类加载器。

此选项可能与J2EE服务器的内置JPA功能冲突。在完整的J2EE环境中，请考虑从JNDI获取`EntityManagerFactory`。或者，在开发者的`LocalContainerEntityManagerFactoryBean`定义中指定一个自定义`persistenceXmlLocation`，例如`META-INF/my-persistence.xml`，并且只在应用程序jar文件中包含有该名称的描述符。因为J2EE服务器仅查找默认的`META-INF/persistence.xml`文件，所以它会忽略这种自定义持久性单元，从而避免了与Spring驱动的JPA设置之间发生冲突。 （例如，这适用于Resin 3.1）

**何时需要加载时间织入？**

并非所有JPA提供商都需要JVM代理。Hibernate就是一个不需要JVM代理的例子。如果开发者的提供商不需要代理或开发者有其他替代方案，例如通过定制编译器或`Ant`任务在构建时应用增强功能，则不用使用加载时间编织器。

`LoadTimeWeaver`是一个Spring提供的接口，它允许以特定方式插入JPA`ClassTransformer`实例，这取决于环境是Web容器还是应用程序服务器。 通过代理挂载`ClassTransformers`通常性能较差。[代理](https://docs.oracle.com/javase/6/docs/api/java/lang/instrument/package-summary.html)会对整个虚拟机进行操作，并检查加载的每个类，这是生产服务器环境中最不需要的额外负载。

Spring为各种环境提供了一些`LoadTimeWeaver`实现，允许`ClassTransformer`实例仅适用于每个类加载器，而不是每个VM。

有关`LoadTimeWeaver`的实现及其设置的通用或定制的各种平台（如Tomcat，WebLogic，GlassFish，Resin和JBoss）的更多了解，请参阅AOP章节中的[Spring配置](http://docs.spring.io/spring/docs/5.0.0.BUILD-SNAPSHOT/spring-framework-reference/htmlsingle/#aop-aj-ltw-spring)一节。

如前面部分所述，开发者可以使用`@EnableLoadTimeWeaving`注解或者`context:load-time-weaver`XML元素来配置上下文范围的`LoadTimeWeaver`。所有JPA `LocalContainerEntityManagerFactoryBeans`都会自动拾取这样的全局织入器。这是设置加载时间织入器的首选方式，为平台（WebLogic，GlassFish，Tomcat，Resin，JBoss或VM代理）提供自动检测功能，并将织入组件自动传播到所有可以感知织入者的Bean：

```java
<context:load-time-weaver/>
<bean id="emf" class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
    ...
</bean>
```

开发者也可以通过`LocalContainerEntityManagerFactoryBean`的`loadTimeWeaver`属性来手动指定专用的织入器：

```java
<bean id="emf" class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
    <property name="loadTimeWeaver">
        <bean class="org.springframework.instrument.classloading.ReflectiveLoadTimeWeaver"/>
    </property>
</bean>
```

无论LTW如何配置，使用这种技术，依赖于仪器的JPA应用程序都可以在目标平台（例如：Tomcat）中运行，而不需要代理。这尤其重要的是当主机应用程序依赖于不同的JPA实现时，因为JPA转换器仅应用于类加载器级，彼此隔离。

#### 处理多个持久化单元 {#处理多个持久化单元}

例如，对于依赖存储在类路径中的各种JARS中的多个持久性单元位置的应用程序，Spring将`PersistenceUnitManager`作为中央仓库来避免可能昂贵的持久性单元发现过程。默认实现允许指定多个位置，这些位置将通过持久性单元名称进行解析并稍后检索。（默认情况下，搜索classpath下的`META-INF/persistence.xml`文件。）

```java
<bean id="pum" class="org.springframework.orm.jpa.persistenceunit.DefaultPersistenceUnitManager">
    <property name="persistenceXmlLocations">
        <list>
            <value>org/springframework/orm/jpa/domain/persistence-multi.xml</value>
            <value>classpath:/my/package/**/custom-persistence.xml</value>
            <value>classpath*:META-INF/persistence.xml</value>
        </list>
    </property>
    <property name="dataSources">
        <map>
            <entry key="localDataSource" value-ref="local-db"/>
            <entry key="remoteDataSource" value-ref="remote-db"/>
        </map>
    </property>
    <!-- if no datasource is specified, use this one -->
    <property name="defaultDataSource" ref="remoteDataSource"/>
</bean>

<bean id="emf" class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
    <property name="persistenceUnitManager" ref="pum"/>
    <property name="persistenceUnitName" value="myCustomUnit"/>
</bean>
```

在默认实现传递给JPA provider之前，是允许通过属性（影响全部持久化单元）或者通过`PersistenceUnitPostProcessor`以编程\(对选择的持久化单元进行\)进行对`PersistenceUnitInfo`进行自定义的。如果没有指定`PersistenceUnitManager`，则由`LocalContainerEntityManagerFactoryBean`在内部创建和使用。

