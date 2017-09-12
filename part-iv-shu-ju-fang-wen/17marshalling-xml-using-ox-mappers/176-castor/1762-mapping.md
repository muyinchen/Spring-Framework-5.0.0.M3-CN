### 17.6.2Mapping

Although it is possible to rely on Castor’s default marshalling behavior, it might be necessary to have more control over it. This can be accomplished using a Castor mapping file. For more information, refer to[Castor XML Mapping](https://castor-data-binding.github.io/castor/reference-guides/1.3.3/html-single/index.html#xml.mapping).

The mapping can be set using the`mappingLocation`resource property, indicated below with a classpath resource.

```java
<beans>
	<bean id="castorMarshaller" class="org.springframework.oxm.castor.CastorMarshaller" >
		<property name="mappingLocation" value="classpath:mapping.xml" />
	</bean>
</beans>
```

#### XML Schema-based Configuration

The`castor-marshaller`tag configures a`org.springframework.oxm.castor.CastorMarshaller`. Here is an example:

```java
<oxm:castor-marshaller id="marshaller" 
    mapping-location="classpath:org/springframework/oxm/castor/mapping.xml"/>
```

The marshaller instance can be configured in two ways, by specifying either the location of a mapping file \(through the`mapping-location`property\), or by identifying Java POJOs \(through the`target-class`or`target-package`properties\) for which there exist corresponding XML descriptor classes. The latter way is usually used in conjunction with XML code generation from XML schemas.

Available attributes are:

| Attribute | Description | Required |
| :--- | :--- | :--- |
| `id` | the id of the marshaller | no |
| `encoding` | the encoding to use for unmarshalling from XML | no |
| `target-class` | a Java class name for a POJO for which an XML class descriptor is available \(as generated through code generation\) | no |
| `target-package` | a Java package name that identifies a package that contains POJOs and their corresponding Castor XML descriptor classes \(as generated through code generation from XML schemas\) | no |
| `mapping-location` | location of a Castor XML mapping file |   |



