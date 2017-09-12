### 17.5.1Jaxb2Marshaller

The`Jaxb2Marshaller`class implements both the Spring`Marshaller`and`Unmarshaller`interface. It requires a context path to operate, which you can set using the`contextPath`property. The context path is a list of colon \(:\) separated Java package names that contain schema derived classes. It also offers a`classesToBeBound`property, which allows you to set an array of classes to be supported by the marshaller. Schema validation is performed by specifying one or more schema resource to the bean, like so:

```java
<beans>
	<bean id="jaxb2Marshaller" class="org.springframework.oxm.jaxb.Jaxb2Marshaller">
		<property name="classesToBeBound">
			<list>
				<value>org.springframework.oxm.jaxb.Flight</value>
				<value>org.springframework.oxm.jaxb.Flights</value>
			</list>
		</property>
		<property name="schema" value="classpath:org/springframework/oxm/schema.xsd"/>
	</bean>

	...

</beans>
```

#### XML Schema-based Configuration

The`jaxb2-marshaller`tag configures a`org.springframework.oxm.jaxb.Jaxb2Marshaller`. Here is an example:

```java
<oxm:jaxb2-marshaller id="marshaller" contextPath="org.springframework.ws.samples.airline.schema"/>
```

Alternatively, the list of classes to bind can be provided to the marshaller via the`class-to-be-bound`child tag:

```java
<oxm:jaxb2-marshaller id="marshaller">
	<oxm:class-to-be-bound name="org.springframework.ws.samples.airline.schema.Airport"/>
	<oxm:class-to-be-bound name="org.springframework.ws.samples.airline.schema.Flight"/>
	...
</oxm:jaxb2-marshaller>
```

Available attributes are:

| Attribute | Description | Required |
| :--- | :--- | :--- |
| `id` | the id of the marshaller | no |
| `contextPath` | the JAXB Context path | no |



