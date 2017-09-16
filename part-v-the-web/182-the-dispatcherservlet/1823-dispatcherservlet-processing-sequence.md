### 18.2.3DispatcherServlet 处理序列

在您设置了`DispatcherServlet`并且针对该特定`DispatcherServlet`启动了一个请求后，`DispatcherServlet`将按如下所示开始处理请求：

* 在请求中搜索并绑定`WebApplicationContext`作为控件和进程中的其他元素可以使用的属性。默认情况下，它将在`DispatcherServlet.WEB_APPLICATION_CONTEXT_ATTRIBUTE`键下绑定。

* 语言环境解析器被绑定到请求，以使进程中的元素能够解决在处理请求时使用的区域设置（渲染视图，准备数据等）。如果您不需要语言环境解析，则不需要它。

* 主题解析器被绑定到使得诸如视图之类的元素确定要使用哪个主题的请求。如果不使用主题，可以忽略它。

* 如果指定了多部分文件解析器，则会检查该请求的多部分;如果找到多部分，则请求被包装在一个`MultipartHttpServletRequest`中，以便进程中的其他元素进一步处理。有关多部分处理的更多信息，请参见[第18.10节“Spring的多部分（文件上传）支持”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-multipart)。

* 搜索适当的处理程序。如果找到处理程序，则执行与处理程序（预处理程序，后处理程序和控制器）关联的执行链，以便准备模型或呈现。

* 如果返回模型，则呈现视图。如果没有返回模型（可能是由于预处理程序或后处理程序拦截请求，可能是出于安全原因），因为请求可能已经被满足，所以不会呈现任何视图。

在`WebApplicationContext`中声明的处理程序异常解析程序在处理请求期间提取异常。使用这些异常解析器允许您定义自定义行为来解决异常。

Spring `DispatcherServlet`还支持返回由Servlet API指定的最后修改日期。确定特定请求的最后修改日期的过程很简单：`DispatcherServlet`查找适当的处理程序映射，并测试发现的处理程序是否实现了`LastModified`接口。如果是，则`LastModified`接口的`long getLastModified(request)`方法的值将返回给客户端。

您可以通过将Servlet初始化参数（`init-param`元素）添加到`web.xml`文件中的Servlet声明来自定义单独的`DispatcherServlet`实例。有关支持的参数列表，请参见下表。

**Table18.2.DispatcherServlet初始化参数**

| Parameter | Explanation |
| :--- | :--- |
| `contextClass` | 实现`WebApplicationContext`的类，它实例化了这个Servlet使用的上下文。 默认情况下，使用`XmlWebApplicationContext`。 |
| `contextConfigLocation` | 传递给上下文实例（由contextClass指定）以指示可以找到上下文的字符串。 该字符串可能包含多个字符串（使用逗号作为分隔符）来支持多个上下文。 在具有两次定义的bean的多个上下文位置的情况下，优先级最高。 |
| `namespace` | `WebApplicationContext`的命名空间。 默认为`[servlet-name] -servlet`。 |



