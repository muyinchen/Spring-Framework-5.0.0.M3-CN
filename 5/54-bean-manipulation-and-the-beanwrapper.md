## 5.4 Bean操作和BeanWrapper

`org.springframework.beans`包遵循Oracle提供的JavaBeans标准。一个JavaBean只是一个包含默认无参构造器的类，它遵循一个命名约定(通过一个例子)：一个名为`bingoMadness`属性将有一个设置方法`setBingoMadness(..)`和一个获取方法`getBingoMadness(..)`。有关JavaBeans和其规范的更多信息，请参考Oracle的网站([javabeans](https://docs.oracle.com/javase/6/docs/api/java/beans/package-summary.html))。

beans包里一个非常重要的类是`BeanWrapper`接口和它的相应实现(`BeanWrapperImpl`)。引用自java文档，`BeanWrapper`提供了设置和获取属性值(单独或批量)、获取属性描述符以及查询属性以确定它们是可读还是可写的功能。`BeanWrapper`还提供对嵌套属性的支持，能够不受嵌套深度的限制启用子属性的属性设置。然后，`BeanWrapper`提供了无需目标类代码的支持就能够添加标准JavaBeans的`PropertyChangeListeners`和`VetoableChangeListeners`的能力。最后然而并非最不重要的是，`BeanWrapper`提供了对索引属性设置的支持。`BeanWrapper`通常不会被应用程序的代码直接使用，而是由`DataBinder`和`BeanFactory`使用。

`BeanWrapper`的名字已经部分暗示了它的工作方式：它包装一个bean以对其执行操作，比如设置和获取属性。