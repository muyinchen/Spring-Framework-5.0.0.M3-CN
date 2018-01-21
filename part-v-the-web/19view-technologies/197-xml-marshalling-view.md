## 19.7 XML Marshalling View

The`MarshallingView`uses an XML`Marshaller`defined in the`org.springframework.oxm`package to render the response content as XML. The object to be marshalled can be set explicitly using``MarhsallingView’s `modelKey``bean property. Alternatively, the view will iterate over all model properties and marshal the first type that is supported by the`Marshaller`. For more information on the functionality in the`org.springframework.oxm`package refer to the chapter[Marshalling XML using O/X Mappers](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/oxm.html).

