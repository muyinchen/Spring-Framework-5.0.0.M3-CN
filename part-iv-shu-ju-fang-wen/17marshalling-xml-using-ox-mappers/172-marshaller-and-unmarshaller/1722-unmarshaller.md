Similar to the`Marshaller`, there is the`org.springframework.oxm.Unmarshaller`interface.

```java
public interface Unmarshaller {

	/**
	 * Unmarshal the given provided Source into an object graph.
	 */
	Object unmarshal(Source source) throws XmlMappingException, IOException;
}
```

This interface also has one method, which reads from the given`javax.xml.transform.Source`\(an XML input abstraction\), and returns the object read. As with Result, Source is a tagging interface that has three concrete implementations. Each wraps a different XML representation, as indicated in the table below.

| Source implementation | Wraps XML representation |
| :--- | :--- |
| `DOMSource` | `org.w3c.dom.Node` |
| `SAXSource` | `org.xml.sax.InputSource`, and`org.xml.sax.XMLReader` |
| `StreamSource` | `java.io.File`,`java.io.InputStream`, or`java.io.Reader` |

Even though there are two separate marshalling interfaces \(`Marshaller`and`Unmarshaller`\), all implementations found in Spring-WS implement both in one class. This means that you can wire up one marshaller class and refer to it both as a marshaller and an unmarshaller in your`applicationContext.xml`.

