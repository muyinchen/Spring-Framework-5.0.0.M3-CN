### 16.3.1在Spring容器中配置SessionFactory

开发者可以将资源如JDBC`DataSource`或Hibernate`SessionFactory`定义为Spring容器中的bean来避免将应用程序对象绑定到硬编码的资源查找上。应用对象需要访问资源的时候，都通过对应的Bean实例进行间接查找，详情可以通过下一节的DAO的定义来参考。

下面引用的应用的XML元数据定义就展示了如何配置JDBC的`DataSource`和`Hibernate`的`SessionFactory`的：

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

这样，从本地的Jaksrta Commons DBCP的`BasicDataSource`转换到JNDI定位的`DataSource`仅仅只需要修改配置文件。

```java
<beans>
    <jee:jndi-lookup id="myDataSource" jndi-name="java:comp/env/jdbc/myds"/>
</beans>
```

You can also access a JNDI-located`SessionFactory`, using Spring’s`JndiObjectFactoryBean`/\`\`to retrieve and expose it. However, that is typically not common outside of an EJB context.

开发者也可以通过Spring的`JndiObjectFactoryBean`或者`<jee:jndi-lookup>`来获取对应Bean以访问JNDI定位的`SessionFactory`。但是，JNDI定位的`SessionFactory`在EJB上下文不常见。

