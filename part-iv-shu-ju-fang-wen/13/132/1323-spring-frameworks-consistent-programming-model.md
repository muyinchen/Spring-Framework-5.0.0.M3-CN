### 13.2.3 Spring Framework’s consistent programming model

Spring resolves the disadvantages of global and local transactions. It enables application developers to use a_consistent\_programming model\_in any environment_. You write your code once, and it can benefit from different transaction management strategies in different environments. The Spring Framework provides both declarative and programmatic transaction management. Most users prefer declarative transaction management, which is recommended in most cases.

With programmatic transaction management, developers work with the Spring Framework transaction abstraction, which can run over any underlying transaction infrastructure. With the preferred declarative model, developers typically write little or no code related to transaction management, and hence do not depend on the Spring Framework transaction API, or any other transaction API.

**Do you need an application server for transaction management?**

The Spring Framework’s transaction management support changes traditional rules as to when an enterprise Java application requires an application server.

In particular, you do not need an application server simply for declarative transactions through EJBs. In fact, even if your application server has powerful JTA capabilities, you may decide that the Spring Framework’s declarative transactions offer more power and a more productive programming model than EJB CMT.

Typically you need an application server’s JTA capability only if your application needs to handle transactions across multiple resources, which is not a requirement for many applications. Many high-end applications use a single, highly scalable database \(such as Oracle RAC\) instead. Standalone transaction managers such as[Atomikos Transactions](http://www.atomikos.com/)and[JOTM](http://jotm.objectweb.org/)are other options. Of course, you may need other application server capabilities such as Java Message Service \(JMS\) and Java EE Connector Architecture \(JCA\).

The Spring Framework_gives you the choice of when to scale your application to a fully loaded application server_. Gone are the days when the only alternative to using EJB CMT or JTA was to write code with local transactions such as those on JDBC connections, and face a hefty rework if you need that code to run within global, container-managed transactions. With the Spring Framework, only some of the bean definitions in your configuration file, rather than your code, need to change.



