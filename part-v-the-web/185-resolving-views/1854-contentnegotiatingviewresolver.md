### 18.5.4ContentNegotiatingViewResolver

`ContentNegotiatingViewResolver`不会自己解析视图，而是委托给其他视图解析器，选择类似于客户端请求的视图。 有两种策略可以让客户从服务器请求表示：

* 对每个资源使用不同的URI，通常在URI中使用不同的文件扩展名。 例如，URI `http://www.example.com/users/fred.pdf`请求用户fred的PDF表示，而`http://www.example.com/users/fred.xml`请求XML表示。

* 为客户端使用相同的URI来定位资源，但是设置`Accept` HTTP请求头来列出它理解的[媒体类型](https://en.wikipedia.org/wiki/Internet_media_type)。 例如，`http://www.example.com/users/fred`的一个HTTP请求的`Accept`标头设置为`application/pdf`，请求用户fred的PDF表示，同时[`http://www.example.com/users/fred`](http://www.example.com/users/fred)使用`Accept`头设置来`text/xml`请求XML表示。 这个策略被称为[内容谈判](https://en.wikipedia.org/wiki/Content_negotiation)。

| ![](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png) |
| :--- |
|  `Accept`头的一个问题是，不可能在HTML中的Web浏览器中设置它。 例如，在Firefox中，它被固定为：`Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8`因此，通常会看到使用 开发基于浏览器的Web应用程序时，每个表示的URI都是不同的。 |

为了支持资源的多重表示，Spring提供了`ContentNegotiatingViewResolver`来根据HTTP请求的文件扩展名或者`Accept`头来解析一个视图。`ContentNegotiatingViewResolver`本身并不执行视图解析，而是委托给你指定的视图解析器列表 bean属性`ViewResolvers`。

`ContentNegotiatingViewResolver`通过将请求媒体类型与`ViewResolver`关联的`View`所支持的媒体类型（也称为`Content-Type`）进行比较来选择适当的`View`来处理请求。 具有兼容的`Content-Type`的列表中的第一个`View`将表示返回给客户端。 如果`ViewResolver`链无法提供兼容的视图，则会查看通过`DefaultViews`属性指定的视图列表。 后面的选项适用于可以呈现当前资源的适当表示的单例视图，而不管逻辑视图的名称如何。 `Accept`头可能包含通配符，例如text/\*，在这种情况下，Content-Type为`text/xml`的`View`是兼容的匹配。

要支持基于文件扩展名的视图的自定义分辨率，请使用`ContentNegotiationManager`：请参见[第18.16.6节“内容协商”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-config-content-negotiation)。

以下是`ContentNegotiatingViewResolver`的配置示例：

```java
<bean class="org.springframework.web.servlet.view.ContentNegotiatingViewResolver">
    <property name="viewResolvers">
        <list>
            <bean class="org.springframework.web.servlet.view.BeanNameViewResolver"/>
            <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
                <property name="prefix" value="/WEB-INF/jsp/"/>
                <property name="suffix" value=".jsp"/>
            </bean>
        </list>
    </property>
    <property name="defaultViews">
        <list>
            <bean class="org.springframework.web.servlet.view.json.MappingJackson2JsonView"/>
        </list>
    </property>
</bean>

<bean id="content" class="com.foo.samples.rest.SampleContentAtomView"/>
```

`InternalResourceViewResolver`处理视图名称和JSP页面的翻译，而`BeanNameViewResolver`则根据bean的名称返回一个视图。 （有关Spring如何查找和实例化视图的更多详细信息，请参阅“[使用ViewResolver界面](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-viewresolver-resolver)解析视图”。）在本例中，内容bean是从`AbstractAtomFeedView`继承的类，它返回Atom RSS提要。 有关创建Atom Feed表示的更多信息，请参阅Atom Views部分。

In the above configuration, if a request is made with an`.html`extension, the view resolver looks for a view that matches the`text/html`media type. The`InternalResourceViewResolver`provides the matching view for`text/html`. If the request is made with the file extension`.atom`, the view resolver looks for a view that matches the`application/atom+xml`media type. This view is provided by the`BeanNameViewResolver`that maps to the`SampleContentAtomView`if the view name returned is`content`. If the request is made with the file extension`.json`, the`MappingJackson2JsonView`instance from the`DefaultViews`list will be selected regardless of the view name. Alternatively, client requests can be made without a file extension but with the`Accept`header set to the preferred media-type, and the same resolution of request to views would occur.

在上面的配置中，如果使用`.html`扩展名进行请求，则视图解析器将查找与`text/html`媒体类型相匹配的视图。 `InternalResourceViewResolver`提供了匹配的视图`fortext/html`。 如果请求是使用文件扩展名`.atom`创建的，则视图解析器会查找与`application/atom+xml`媒体类型相匹配的视图。 如果返回的视图名称是内容，则此视图由映射到`SampleContentAtomView`的`BeanNameViewResolver`提供。 如果请求是使用文件扩展名`.json`创建的，则无论视图名称如何，都会选择`DefaultViews`列表中的`MappingJackson2JsonView`实例。 或者，客户端请求可以在没有文件扩展名的情况下进行，但将`Accept`头设置为首选的媒体类型，并且对视图的请求的分辨率也会发生。

| ![](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png) |
| :--- |
| 如果“ContentNegotiatingViewResolver”的ViewResolver列表未被明确配置，它会自动使用应用程序上下文中定义的任何ViewResolvers。 |

下面显示了相应的控制器代码，该代码返回Atom RSS提要，其格式为`http://localhost/content.atom`或`http://localhost/content`，并带有application/atom+xml的`Accept`头。

```java
@Controller
public class ContentController {

    private List<SampleContent> contentList = new ArrayList<SampleContent>();

    @GetMapping("/content")
    public ModelAndView getContent() {
        ModelAndView mav = new ModelAndView();
        mav.setViewName("content");
        mav.addObject("sampleContentList", contentList);
        return mav;
    }

}
```



