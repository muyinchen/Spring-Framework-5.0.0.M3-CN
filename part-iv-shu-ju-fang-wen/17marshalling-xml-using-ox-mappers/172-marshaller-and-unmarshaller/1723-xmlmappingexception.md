### 17.2.3XmlMappingException

Spring converts exceptions from the underlying O/X mapping tool to its own exception hierarchy with the`XmlMappingException`as the root exception. As can be expected, these runtime exceptions wrap the original exception so no information will be lost.

Additionally, the`MarshallingFailureException`and`UnmarshallingFailureException`provide a distinction between marshalling and unmarshalling operations, even though the underlying O/X mapping tool does not do so.

The O/X Mapping exception hierarchy is shown in the following figure:

![](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/oxm-exceptions.png)

O/X Mapping exception hierarchy

