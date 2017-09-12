### 17.2.1 Marshaller

Spring abstracts all marshalling operations behind the`org.springframework.oxm.Marshaller`interface, the main method of which is shown below.

```java
public interface Marshaller {

	/**
	 * Marshal the object graph with the given root into the provided Result.
	 */
	void marshal(Object graph, Result result) throws XmlMappingException, IOException;
}
```

The`Marshaller`interface has one main method, which marshals the given object to a given`javax.xml.transform.Result`. Result is a tagging interface that basically represents an XML output abstraction: concrete implementations wrap various XML representations, as indicated in the table below.

| Result implementation | Wraps XML representation |
| :--- | :--- |
| `DOMResult` | `org.w3c.dom.Node` |
| `SAXResult` | `org.xml.sax.ContentHandler` |
| `StreamResult` | `java.io.File`,`java.io.OutputStream`, or`java.io.Writer` |

| ![](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png) |
| :--- |
| Although the`marshal()`method accepts a plain object as its first parameter, most`Marshaller`implementations cannot handle arbitrary objects. Instead, an object class must be mapped in a mapping file, marked with an annotation, registered with the marshaller, or have a common base class. Refer to the further sections in this chapter to determine how your O/X technology of choice manages this. |



