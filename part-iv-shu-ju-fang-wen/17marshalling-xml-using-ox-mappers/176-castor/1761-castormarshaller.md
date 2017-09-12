### 17.6.1CastorMarshaller

As with JAXB, the`CastorMarshaller`implements both the`Marshaller`and`Unmarshaller`interface. It can be wired up as follows:

```java
<beans>
	<bean id="castorMarshaller" class="org.springframework.oxm.castor.CastorMarshaller" />
	...
</beans>
```



