### 13.5.9Using @Transactional with AspectJ

It is also possible to use the Spring Framework’s`@Transactional`support outside of a Spring container by means of an AspectJ aspect. To do so, you first annotate your classes \(and optionally your classes' methods\) with the`@Transactional`annotation, and then you link \(weave\) your application with the`org.springframework.transaction.aspectj.AnnotationTransactionAspect`defined in the`spring-aspects.jar`file. The aspect must also be configured with a transaction manager. You can of course use the Spring Framework’s IoC container to take care of dependency-injecting the aspect. The simplest way to configure the transaction management aspect is to use the```element and specify the``mode`attribute to`aspectj\`as described in[Section13.5.6, “Using @Transactional”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/transaction.html#transaction-declarative-annotations). Because we’re focusing here on applications running outside of a Spring container, we’ll show you how to do it programmatically.

| ![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| :--- |
| Prior to continuing, you may want to read[Section13.5.6, “Using @Transactional”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/transaction.html#transaction-declarative-annotations)and[Chapter7,_Aspect Oriented Programming with Spring_](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/aop.html)respectively. |

```java
// construct an appropriate transaction manager
DataSourceTransactionManager txManager = new DataSourceTransactionManager(getDataSource());

// configure the AnnotationTransactionAspect to use it; this must be done before executing any transactional methods
AnnotationTransactionAspect.aspectOf().setTransactionManager(txManager);
```

| ![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| :--- |
| When using this aspect, you must annotate the_implementation_class \(and/or methods within that class\),_not_the interface \(if any\) that the class implements. AspectJ follows Java’s rule that annotations on interfaces are_not inherited_. |

The`@Transactional`annotation on a class specifies the default transaction semantics for the execution of any method in the class.

The`@Transactional`annotation on a method within the class overrides the default transaction semantics given by the class annotation \(if present\). Any method may be annotated, regardless of visibility.

To weave your applications with the`AnnotationTransactionAspect`you must either build your application with AspectJ \(see the[AspectJ Development Guide](https://www.eclipse.org/aspectj/doc/released/devguide/index.html)\) or use load-time weaving. See[Section7.8.4, “Load-time weaving with AspectJ in the Spring Framework”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/aop.html#aop-aj-ltw)for a discussion of load-time weaving with AspectJ.

