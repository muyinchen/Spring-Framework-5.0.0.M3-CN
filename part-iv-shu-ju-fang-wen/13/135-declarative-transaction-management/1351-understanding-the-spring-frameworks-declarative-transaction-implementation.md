### 13.5.1Understanding the Spring Framework’s declarative transaction implementation

It is not sufficient to tell you simply to annotate your classes with the`@Transactional`annotation, add`@EnableTransactionManagement`to your configuration, and then expect you to understand how it all works. This section explains the inner workings of the Spring Framework’s declarative transaction infrastructure in the event of transaction-related issues.

The most important concepts to grasp with regard to the Spring Framework’s declarative transaction support are that this support is enabled[_via AOP proxies_](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/aop.html#aop-understanding-aop-proxies), and that the transactional advice is driven by_metadata_\(currently XML- or annotation-based\). The combination of AOP with transactional metadata yields an AOP proxy that uses a`TransactionInterceptor`in conjunction with an appropriate`PlatformTransactionManager`implementation to drive transactions_around method invocations_.

| ![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png) |
| :--- |
| Spring AOP is covered in[Chapter7,_Aspect Oriented Programming with Spring_](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/aop.html). |

Conceptually, calling a method on a transactional proxy looks like this…​

![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/tx.png "tx")

