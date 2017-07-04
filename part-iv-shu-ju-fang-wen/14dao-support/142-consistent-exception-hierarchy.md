## 14.2Consistent exception hierarchy

Spring provides a convenient translation from technology-specific exceptions like`SQLException`to its own exception class hierarchy with the`DataAccessException`as the root exception. These exceptions wrap the original exception so there is never any risk that one might lose any information as to what might have gone wrong.

In addition to JDBC exceptions, Spring can also wrap Hibernate-specific exceptions, converting them to a set of focused runtime exceptions \(the same is true for JPA exceptions\). This allows one to handle most persistence exceptions, which are non-recoverable, only in the appropriate layers, without having annoying boilerplate catch-and-throw blocks and exception declarations in one’s DAOs. \(One can still trap and handle exceptions anywhere one needs to though.\) As mentioned above, JDBC exceptions \(including database-specific dialects\) are also converted to the same hierarchy, meaning that one can perform some operations with JDBC within a consistent programming model.

The above holds true for the various template classes in Springs support for various ORM frameworks. If one uses the interceptor-based classes then the application must care about handling`HibernateExceptions`and`PersistenceExceptions`itself, preferably via delegating to`SessionFactoryUtils’`convertHibernateAccessException\(..\)`or`convertJpaAccessException\(\)`methods respectively. These methods convert the exceptions to ones that are compatible with the exceptions in the`org.springframework.dao`exception hierarchy. As`PersistenceExceptions\`are unchecked, they can simply get thrown too, sacrificing generic DAO abstraction in terms of exceptions though.

The exception hierarchy that Spring provides can be seen below. \(Please note that the class hierarchy detailed in the image shows only a subset of the entire`DataAccessException`hierarchy.\)

![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/DataAccessException.gif)

  




