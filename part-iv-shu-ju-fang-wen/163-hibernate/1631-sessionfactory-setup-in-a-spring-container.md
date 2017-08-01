### 16.3.1SessionFactory setup in a Spring container

To avoid tying application objects to hard-coded resource lookups, you can define resources such as a JDBC`DataSource`or a Hibernate`SessionFactory`as beans in the Spring container. Application objects that need to access resources receive references to such predefined instances through bean references, as illustrated in the DAO definition in the next section.

The following excerpt from an XML application context definition shows how to set up a JDBC`DataSource`and a Hibernate`SessionFactory`on top of it:

```java
<beans>

	<bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
		<property name="driverClassName" value="org.hsqldb.jdbcDriver"/>
		<property name="url" value="jdbc:hsqldb:hsql://localhost:9001"/>
		<property name="username" value="sa"/>
		<property name="password" value=""/>
	</bean>

	<bean id="mySessionFactory" class="org.springframework.orm.hibernate5.LocalSessionFactoryBean">
		<property name="dataSource" ref="myDataSource"/>
		<property name="mappingResources">
			<list>
				<value>product.hbm.xml</value>
			</list>
		</property>
		<property name="hibernateProperties">
			<value>
				hibernate.dialect=org.hibernate.dialect.HSQLDialect
			</value>
		</property>
	</bean>

</beans>
```

Switching from a local Jakarta Commons DBCP`BasicDataSource`to a JNDI-located`DataSource`\(usually managed by an application server\) is just a matter of configuration:

```java
<beans>
	<jee:jndi-lookup id="myDataSource" jndi-name="java:comp/env/jdbc/myds"/>
</beans>
```

You can also access a JNDI-located`SessionFactory`, using Spring’s`JndiObjectFactoryBean`/\`\`to retrieve and expose it. However, that is typically not common outside of an EJB context.

