## 19.5 JSP & JSTL

Spring provides a couple of out-of-the-box solutions for JSP and JSTL views. Using JSP or JSTL is done using a normal view resolver defined in the`WebApplicationContext`. Furthermore, of course you need to write some JSPs that will actually render the view.

| ![](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png "\[Note\]") |
| :--- |
| Setting up your application to use JSTL is a common source of error, mainly caused by confusion over the different servlet spec., JSP and JSTL version numbers, what they mean and how to declare the taglibs correctly. The article[How to Reference and Use JSTL in your Web Application](http://www.mularien.com/blog/2008/04/24/how-to-reference-and-use-jstl-in-your-web-application/)provides a useful guide to the common pitfalls and how to avoid them. Note that as of Spring 3.0, the minimum supported servlet version is 2.4 \(JSP 2.0 and JSTL 1.1\), which reduces the scope for confusion somewhat. |



