### 18.5.1使用ViewResolver界面解析视图

如[第18.3节，“实施控制器”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-controller)中所讨论的，Spring Web MVC控制器中的所有处理器方法必须明确地（例如，通过返回`String`，`View`或`ModelAndView`）或隐式地（即基于惯例）。 Spring中的视图由逻辑视图名称来解决，并由视图解析器解决。 Spring有不少解析器。 这个表格列出了其中的大部分; 下面有几个例子。

**Table18.3.查看解析器**

| 视图解析器 | 描述 |
| :--- | :--- |
| `AbstractCachingViewResolver` | 抽象视图解析器缓存视图。通常情况下，需要准备才能使用; 扩展此视图解析器提供缓存。 |
| `XmlViewResolver` | `ViewResolver`的实现，它接受一个用XML编写的配置文件，其中使用与Spring的XML bean工厂相同的DTD。 默认配置文件是`/WEB-INF/views.xml`。 |
| `ResourceBundleViewResolver` | `ViewResolver`的实现，它使用由bundle基名指定的`ResourceBundle`中的bean定义。 通常，您可以在位于类路径中的属性文件中定义该包。 默认文件名是`views.properties`。 |
| `UrlBasedViewResolver` | `ViewResolver`接口的简单实现，它实现了逻辑视图名称直接解析到URL，而没有明确的映射定义。 如果您的逻辑名称以直观的方式与您的视图资源的名称相匹配，而不需要任意映射，则这是适当的。 |
| `InternalResourceViewResolver` | `UrlBasedViewResolver`的便捷子类支持`InternalResourceView`（实际上是Servlet和JSP）和子类（如`JstlView`和`TilesView`）。 您可以使用`setViewClass(..)`为由此解析器生成的所有视图指定视图类。 有关详细信息，请参阅`UrlBasedViewResolver` javadocs。 |
| `FreeMarkerViewResolver` | `UrlBasedViewResolver`方便的子类，支持`FreeMarkerView`和它们的自定义子类。 |
| `ContentNegotiatingViewResolver` | 实现基于请求文件名或`Accept`头来解析视图的`ViewResolver`接口。 请参见[第18.5.4节“ContentNegotiatingViewResolver”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-multiple-representations)。 |

例如，使用JSP作为视图技术，可以使用`UrlBasedViewResolver`。此视图解析器将视图名称转换为URL，并将请求转交给`RequestDispatcher`以呈现视图。

```java
<bean id="viewResolver"
        class="org.springframework.web.servlet.view.UrlBasedViewResolver">
    <property name="viewClass" value="org.springframework.web.servlet.view.JstlView"/>
    <property name="prefix" value="/WEB-INF/jsp/"/>
    <property name="suffix" value=".jsp"/>
</bean>
```

当将`test`作为逻辑视图名称返回时，此视图解析器将请求转发给`RequestDispatcher`，`RequestDispatcher`将请求发送到`/WEB-INF/jsp/test.jsp`。

当您在Web应用程序中结合不同的视图技术时，可以使用`ResourceBundleViewResolver`：

```java
<bean id="viewResolver"
        class="org.springframework.web.servlet.view.ResourceBundleViewResolver">
    <property name="basename" value="views"/>
    <property name="defaultParentView" value="parentView"/>
</bean>
```

`ResourceBundleViewResolver`检查由基本名称标识的`ResourceBundle`，对于它应该解析的每个视图，它使用属性`[viewname].(class)`作为视图类的值和属性`[viewname].url`的值作为 视图URL。 范例可以在下一章讨论视图技术。 正如你所看到的，你可以识别一个父视图，属性文件中的所有视图都从其中“扩展”。 例如，这样你可以指定一个默认的视图类。

| ![](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png) |
| :--- |
| `AbstractCachingViewResolver`的子类缓存查看它们解析的实例。 缓存提高了某些视图技术的性能。 可以通过将`cache`属性设置为`false`来关闭缓存。 此外，如果您必须在运行时刷新某个视图（例如，在修改FreeMarker模板时），则可以使用`removeFromCache(String viewName, Locale loc)`方法。 |



