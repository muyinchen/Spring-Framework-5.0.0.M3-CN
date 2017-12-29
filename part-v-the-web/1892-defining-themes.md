### 18.9.2定义主题

要在您的Web应用程序中使用主题，您必须设置`org.springframework.ui.context.ThemeSource`接口的实现。 `WebApplicationContext`接口扩展了`ThemeSource`，但将其职责委托给专用的实现。 默认情况下，委托将是一个`org.springframework.ui.context.support.ResourceBundleThemeSource`实现，它从类路径的根目录加载属性文件。 要使用自定义的`ThemeSource`实现或配置`ResourceBundleThemeSource`的基本名称前缀，可以在应用程序上下文中使用保留名称`themeSource`注册一个bean。 Web应用程序上下文自动检测具有该名称的bean并使用它。

使用`ResourceBundleThemeSource`时，主题是在一个简单的属性文件中定义的。 属性文件列出组成主题的资源。 这里是一个例子：

```java
styleSheet=/themes/cool/style.css
background=/themes/cool/img/coolBg.jpg
```

属性的键是从视图代码引用主题元素的名称。 对于JSP，通常使用`spring：theme`自定义标签来实现，这与`spring：message`标签非常相似。 以下JSP片段使用前面示例中定义的主题来自定义外观：

```java
<%@ taglib prefix="spring" uri="http://www.springframework.org/tags"%>
<html>
    <head>
        <link rel="stylesheet" href="<spring:theme code='styleSheet'/>" type="text/css"/>
    </head>
    <body style="background=<spring:theme code='background'/>">
        ...
    </body>
</html>
```

默认情况下，`ResourceBundleThemeSource`使用空的基本名称前缀。 因此，属性文件是从类路径的根目录加载的。 因此，您可以将`cool.properties`主题定义放在类路径根目录中，例如`/WEB-INF/classes`中。 `ResourceBundleThemeSource`使用标准的Java资源包加载机制，允许主题完全国际化。 例如，我们可以有一个`/WEB-INF/classes/cool_nl.properties`引用一个带有荷兰文字的特殊背景图片。

