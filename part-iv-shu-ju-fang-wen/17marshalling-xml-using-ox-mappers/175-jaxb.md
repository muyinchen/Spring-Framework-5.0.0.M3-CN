## 17.5 JAXB

The JAXB binding compiler translates a W3C XML Schema into one or more Java classes, a`jaxb.properties`file, and possibly some resource files. JAXB also offers a way to generate a schema from annotated Java classes.

Spring supports the JAXB 2.0 API as XML marshalling strategies, following the`Marshaller`and`Unmarshaller`interfaces described in[Section 17.2, “Marshaller and Unmarshaller”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/oxm.html#oxm-marshaller-unmarshaller). The corresponding integration classes reside in the`org.springframework.oxm.jaxb`package.

