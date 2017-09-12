## 17.4XML Schema-based Configuration

Marshallers could be configured more concisely using tags from the OXM namespace. To make these tags available, the appropriate schema has to be referenced first in the preamble of the XML configuration file. Note the 'oxm' related text below:

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:oxm="http://www.springframework.org/schema/oxm" 
	xsi:schemaLocation="http://www.springframework.org/schema/beans 
	http://www.springframework.org/schema/beans/spring-beans.xsd 
	http://www.springframework.org/schema/oxm 
	http://www.springframework.org/schema/oxm/spring-oxm.xsd">
```

Currently, the following tags are available:

* [`jaxb2-marshaller`](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/oxm.html#oxm-jaxb2-xsd)

* [`jibx-marshaller`](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/oxm.html#oxm-jibx-xsd)

* [`castor-marshaller`](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/oxm.html#oxm-castor-xsd)

Each tag will be explained in its respective marshaller’s section. As an example though, here is how the configuration of a JAXB2 marshaller might look like:

```java
<oxm:jaxb2-marshaller id="marshaller" contextPath="org.springframework.ws.samples.airline.schema"/>
```



