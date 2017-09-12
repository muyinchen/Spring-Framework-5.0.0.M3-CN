### 17.1.3 Consistent Exception Hierarchy

Spring provides a conversion from exceptions from the underlying O/X mapping tool to its own exception hierarchy with the`XmlMappingException`as the root exception. As can be expected, these runtime exceptions wrap the original exception so no information is lost.

