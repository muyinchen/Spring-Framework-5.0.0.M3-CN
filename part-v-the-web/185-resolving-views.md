## 18.5 Resolving views

All MVC frameworks for web applications provide a way to address views. Spring provides view resolvers, which enable you to render models in a browser without tying you to a specific view technology. Out of the box, Spring enables you to use JSPs, FreeMarker templates and XSLT views, for example. See[Chapter 19,View technologies](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/view.html)for a discussion of how to integrate and use a number of disparate view technologies.

The two interfaces that are important to the way Spring handles views are`ViewResolver`and`View`. The`ViewResolver`provides a mapping between view names and actual views. The`View`interface addresses the preparation of the request and hands the request over to one of the view technologies.

