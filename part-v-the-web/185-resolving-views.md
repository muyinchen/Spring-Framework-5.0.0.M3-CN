## 18.5 解决观点

所有用于Web应用程序的MVC框架都提供了一种处理视图的方法。 Spring提供了视图解析器，它使您能够在浏览器中渲染模型，而不需要将您视为特定的视图技术。开箱即用，例如，Spring允许您使用JSP，FreeMarker模板和XSLT视图。请参见[第19章，查看技术](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/view.html)对于如何整合并使用不同的视图技术的讨论。

对于Spring处理视图的方式来说，两个重要的接口是`ViewResolver`和`View`。 `ViewResolver`提供了视图名称和实际视图之间的映射。 `View`接口处理请求的准备，并将请求交给其中一种视图技术。

