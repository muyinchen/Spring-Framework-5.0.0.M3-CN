## 16.3 Hibernate

我们将首先介绍Spring环境中的[Hibernate 5](http://hibernate.org/)，然后通过使用Hibernate 5来演示Spring集成O/R映射器的方法。本节将详细介绍许多问题，并显示DAO实现和事务划分的不同变体。这些模式中大多数可以直接转换为所有其他支持的ORM工具。本章中的以下部分将通过简单的例子来介绍其他ORM技术。

| ![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png.pagespeed.ce.9zQ_1wVwzR.png "\[Note\]") |
| :--- |
| 从Spring 5.0开始，Spring需要Hibernate ORM对JPA的支持要基于4.3或更高的版本，甚至Hibernate ORM 5.0+可以针对本机Hibernate Session API进行编程。请注意，Hibernate团队可能不会在5.0之前维护任何版本，仅仅专注于5.2以后的版本。  |



