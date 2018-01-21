## 19.13 XML Mapping View

The`MappingJackson2XmlView`uses the[Jackson XML extension](https://github.com/FasterXML/jackson-dataformat-xml)'s`XmlMapper`to render the response content as XML. If the model contains multiples entries, the object to be serialized should be set explicitly using the`modelKey`bean property. If the model contains a single entry, it will be serialized automatically.

XML mapping can be customized as needed through the use of JAXB or Jackson’s provided annotations. When further control is needed, a custom`XmlMapper`can be injected through the`ObjectMapper`property for cases where custom XML serializers/deserializers need to be provided for specific types.

