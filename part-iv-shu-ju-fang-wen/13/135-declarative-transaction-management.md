## 13.5Declarative transaction management

| ![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png) |
| :--- |
| Most Spring Framework users choose declarative transaction management. This option has the least impact on application code, and hence is most consistent with the ideals of a_non-invasive_lightweight container. |

The Spring Framework’s declarative transaction management is made possible with Spring aspect-oriented programming \(AOP\), although, as the transactional aspects code comes with the Spring Framework distribution and may be used in a boilerplate fashion, AOP concepts do not generally have to be understood to make effective use of this code.

The Spring Framework’s declarative transaction management is similar to EJB CMT in that you can specify transaction behavior \(or lack of it\) down to individual method level. It is possible to make a`setRollbackOnly()`call within a transaction context if necessary. The differences between the two types of transaction management are:

* Unlike EJB CMT, which is tied to JTA, the Spring Framework’s declarative transaction management works in any environment. It can work with JTA transactions or local transactions using JDBC, JPA or Hibernate by simply adjusting the configuration files.

* You can apply the Spring Framework declarative transaction management to any class, not merely special classes such as EJBs.

* The Spring Framework offers declarative[_rollback rules_,](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/transaction.html#transaction-declarative-rolling-back)a feature with no EJB equivalent. Both programmatic and declarative support for rollback rules is provided.

* The Spring Framework enables you to customize transactional behavior, by using AOP. For example, you can insert custom behavior in the case of transaction rollback. You can also add arbitrary advice, along with the transactional advice. With EJB CMT, you cannot influence the container’s transaction management except with`setRollbackOnly()`.

* The Spring Framework does not support propagation of transaction contexts across remote calls, as do high-end application servers. If you need this feature, we recommend that you use EJB. However, consider carefully before using such a feature, because normally, one does not want transactions to span remote calls.

**Where is TransactionProxyFactoryBean?**

Declarative transaction configuration in versions of Spring 2.0 and above differs considerably from previous versions of Spring. The main difference is that there is no longer any need to configure`TransactionProxyFactoryBean`beans.

The pre-Spring 2.0 configuration style is still 100% valid configuration; think of the new```as simply defining``TransactionProxyFactoryBean\`beans on your behalf.

The concept of rollback rules is important: they enable you to specify which exceptions \(and throwables\) should cause automatic rollback. You specify this declaratively, in configuration, not in Java code. So, although you can still call`setRollbackOnly()`on the`TransactionStatus`object to roll back the current transaction back, most often you can specify a rule that`MyApplicationException`must always result in rollback. The significant advantage to this option is that business objects do not depend on the transaction infrastructure. For example, they typically do not need to import Spring transaction APIs or other Spring APIs.

Although EJB container default behavior automatically rolls back the transaction on a_system exception_\(usually a runtime exception\), EJB CMT does not roll back the transaction automatically on an_application exception_\(that is, a checked exception other than`java.rmi.RemoteException`\). While the Spring default behavior for declarative transaction management follows EJB convention \(roll back is automatic only on unchecked exceptions\), it is often useful to customize this behavior.



