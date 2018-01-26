## 19.5 JSP & JSTL

Spring为JSP和JSTL视图提供了一些开箱即用的解决方案。 使用JSP或JSTL是使用`WebApplicationContext`中定义的普通视图解析器完成的。 此外，当然你需要编写一些实际呈现视图的JSP。

| ![](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png "\[Note\]") |
| :--- |
| 设置你的应用程序使用JSTL是一个常见的错误来源，主要是由于对不同的servlet规范，JSP和JSTL版本号，它们是什么意思以及如何正确地声明taglibs造成混淆。 文章[How to Reference and Use JSTL in your Web Application](http://www.mularien.com/blog/2008/04/24/how-to-reference-and-use-jstl-in-your-web-application/)提供了常见陷阱的有用指南以及如何避免它们。 请注意，从Spring 3.0开始，受支持的最小servlet版本是2.4（JSP 2.0和JSTL 1.1），这在一定程度上减少了混淆的范围。 |



