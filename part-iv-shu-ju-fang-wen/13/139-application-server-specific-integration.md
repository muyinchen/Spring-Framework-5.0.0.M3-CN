## 13.9Application server-specific integration

Spring’s transaction abstraction generally is application server agnostic. Additionally, Spring’s`JtaTransactionManager`class, which can optionally perform a JNDI lookup for the JTA`UserTransaction`and`TransactionManager`objects, autodetects the location for the latter object, which varies by application server. Having access to the JTA`TransactionManager`allows for enhanced transaction semantics, in particular supporting transaction suspension. See the`JtaTransactionManager`javadocs for details.

Spring’s`JtaTransactionManager`is the standard choice to run on Java EE application servers, and is known to work on all common servers. Advanced functionality such as transaction suspension works on many servers as well — including GlassFish, JBoss and Geronimo — without any special configuration required. However, for fully supported transaction suspension and further advanced integration, Spring ships special adapters for WebLogic Server and WebSphere. These adapters are discussed in the following sections.

_For standard scenarios, including WebLogic Server and WebSphere, consider using the convenientconfiguration element._When configured, this element automatically detects the underlying server and chooses the best transaction manager available for the platform. This means that you won’t have to configure server-specific adapter classes \(as discussed in the following sections\) explicitly; rather, they are chosen automatically, with the standard`JtaTransactionManager`as default fallback.

