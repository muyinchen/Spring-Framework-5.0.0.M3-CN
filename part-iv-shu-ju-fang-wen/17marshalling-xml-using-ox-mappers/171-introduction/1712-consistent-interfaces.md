### 17.1.2 Consistent Interfaces

Spring’s O/X mapping operates through two global interfaces: the`Marshaller`and`Unmarshaller`interface. These abstractions allow you to switch O/X mapping frameworks with relative ease, with little or no changes required on the classes that do the marshalling. This approach has the additional benefit of making it possible to do XML marshalling with a mix-and-match approach \(e.g. some marshalling performed using JAXB, other using Castor\) in a non-intrusive fashion, leveraging the strength of each technology.

