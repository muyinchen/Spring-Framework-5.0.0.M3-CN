### 16.4.1Three options for JPA setup in a Spring environment

The Spring JPA support offers three ways of setting up the JPA`EntityManagerFactory`that will be used by the application to obtain an entity manager.

#### LocalEntityManagerFactoryBean

| ![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| :--- |
| Only use this option in simple deployment environments such as stand-alone applications and integration tests. |

The`LocalEntityManagerFactoryBean`creates an`EntityManagerFactory`suitable for simple deployment environments where the application uses only JPA for data access. The factory bean uses the JPA`PersistenceProvider`autodetection mechanism \(according to JPA’s Java SE bootstrapping\) and, in most cases, requires you to specify only the persistence unit name:

```java
<beans>
	<bean id="myEmf" class="org.springframework.orm.jpa.LocalEntityManagerFactoryBean">
		<property name="persistenceUnitName" value="myPersistenceUnit"/>
	</bean>
</beans>
```

This form of JPA deployment is the simplest and the most limited. You cannot refer to an existing JDBC`DataSource`bean definition and no support for global transactions exists. Furthermore, weaving \(byte-code transformation\) of persistent classes is provider-specific, often requiring a specific JVM agent to specified on startup. This option is sufficient only for stand-alone applications and test environments, for which the JPA specification is designed.

#### Obtaining an EntityManagerFactory from JNDI

| ![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| :--- |
| Use this option when deploying to a Java EE server. Check your server’s documentation on how to deploy a custom JPA provider into your server, allowing for a different provider than the server’s default. |

Obtaining an`EntityManagerFactory`from JNDI \(for example in a Java EE environment\), is simply a matter of changing the XML configuration:

```java
<beans>
	<jee:jndi-lookup id="myEmf" jndi-name="persistence/myPersistenceUnit"/>
</beans>
```

This action assumes standard Java EE bootstrapping: the Java EE server autodetects persistence units \(in effect,`META-INF/persistence.xml`files in application jars\) and`persistence-unit-ref`entries in the Java EE deployment descriptor \(for example,`web.xml`\) and defines environment naming context locations for those persistence units.

In such a scenario, the entire persistence unit deployment, including the weaving \(byte-code transformation\) of persistent classes, is up to the Java EE server. The JDBC`DataSource`is defined through a JNDI location in the`META-INF/persistence.xml`file; EntityManager transactions are integrated with the server’s JTA subsystem. Spring merely uses the obtained`EntityManagerFactory`, passing it on to application objects through dependency injection, and managing transactions for the persistence unit, typically through`JtaTransactionManager`.

If multiple persistence units are used in the same application, the bean names of such JNDI-retrieved persistence units should match the persistence unit names that the application uses to refer to them, for example, in`@PersistenceUnit`and`@PersistenceContext`annotations.

#### LocalContainerEntityManagerFactoryBean

| ![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| :--- |
| Use this option for full JPA capabilities in a Spring-based application environment. This includes web containers such as Tomcat as well as stand-alone applications and integration tests with sophisticated persistence requirements. |

The`LocalContainerEntityManagerFactoryBean`gives full control over`EntityManagerFactory`configuration and is appropriate for environments where fine-grained customization is required. The`LocalContainerEntityManagerFactoryBean`creates a`PersistenceUnitInfo`instance based on the`persistence.xml`file, the supplied`dataSourceLookup`strategy, and the specified`loadTimeWeaver`. It is thus possible to work with custom data sources outside of JNDI and to control the weaving process. The following example shows a typical bean definition for a`LocalContainerEntityManagerFactoryBean`:

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

The following example shows a typical`persistence.xml`file:

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
| The```shortcut indicates that*no*scanning for annotated entity classes is supposed to occur. An explicit 'true' value specified -``true`- also means no scan.`false`does trigger a scan; however, it is recommended to simply omit the`exclude-unlisted-classes\`element if you want entity class scanning to occur. |

Using the`LocalContainerEntityManagerFactoryBean`is the most powerful JPA setup option, allowing for flexible local configuration within the application. It supports links to an existing JDBC`DataSource`, supports both local and global transactions, and so on. However, it also imposes requirements on the runtime environment, such as the availability of a weaving-capable class loader if the persistence provider demands byte-code transformation.

This option may conflict with the built-in JPA capabilities of a Java EE server. In a full Java EE environment, consider obtaining your`EntityManagerFactory`from JNDI. Alternatively, specify a custom`persistenceXmlLocation`on your`LocalContainerEntityManagerFactoryBean`definition, for example, META-INF/my-persistence.xml, and only include a descriptor with that name in your application jar files. Because the Java EE server only looks for default`META-INF/persistence.xml`files, it ignores such custom persistence units and hence avoid conflicts with a Spring-driven JPA setup upfront. \(This applies to Resin 3.1, for example.\)

**When is load-time weaving required?**

Not all JPA providers require a JVM agent. Hibernate is an example of one that does not. If your provider does not require an agent or you have other alternatives, such as applying enhancements at build time through a custom compiler or an ant task, the load-time weaver_should not_be used.

The`LoadTimeWeaver`interface is a Spring-provided class that allows JPA`ClassTransformer`instances to be plugged in a specific manner, depending whether the environment is a web container or application server. Hooking`ClassTransformers`through an[agent](https://docs.oracle.com/javase/6/docs/api/java/lang/instrument/package-summary.html)typically is not efficient. The agents work against the_entire virtual machine_and inspect_every_class that is loaded, which is usually undesirable in a production server environment.

Spring provides a number of`LoadTimeWeaver`implementations for various environments, allowing`ClassTransformer`instances to be applied only_per class loader_and not per VM.

Refer to[the section called “Spring configuration”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/aop.html#aop-aj-ltw-spring)in the AOP chapter for more insight regarding the`LoadTimeWeaver`implementations and their setup, either generic or customized to various platforms \(such as Tomcat, WebLogic, GlassFish, Resin and JBoss\).

As described in the aforementioned section, you can configure a context-wide`LoadTimeWeaver`using the`@EnableLoadTimeWeaving`annotation of`context:load-time-weaver`XML element. Such a global weaver is picked up by all JPA`LocalContainerEntityManagerFactoryBeans`automatically. This is the preferred way of setting up a load-time weaver, delivering autodetection of the platform \(WebLogic, GlassFish, Tomcat, Resin, JBoss or VM agent\) and automatic propagation of the weaver to all weaver-aware beans:

```java
<context:load-time-weaver/>
<bean id="emf" class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
	...
</bean>
```

However, if needed, one can manually specify a dedicated weaver through the`loadTimeWeaver`property:

```java
<bean id="emf" class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
	<property name="loadTimeWeaver">
		<bean class="org.springframework.instrument.classloading.ReflectiveLoadTimeWeaver"/>
	</property>
</bean>
```

No matter how the LTW is configured, using this technique, JPA applications relying on instrumentation can run in the target platform \(ex: Tomcat\) without needing an agent. This is important especially when the hosting applications rely on different JPA implementations because the JPA transformers are applied only at class loader level and thus are isolated from each other.

#### Dealing with multiple persistence units

For applications that rely on multiple persistence units locations, stored in various JARS in the classpath, for example, Spring offers the`PersistenceUnitManager`to act as a central repository and to avoid the persistence units discovery process, which can be expensive. The default implementation allows multiple locations to be specified that are parsed and later retrieved through the persistence unit name. \(By default, the classpath is searched for`META-INF/persistence.xml`files.\)

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

The default implementation allows customization of the`PersistenceUnitInfo`instances, before they are fed to the JPA provider, declaratively through its properties, which affect_all_hosted units, or programmatically, through the`PersistenceUnitPostProcessor`, which allows persistence unit selection. If no`PersistenceUnitManager`is specified, one is created and used internally by`LocalContainerEntityManagerFactoryBean`.

