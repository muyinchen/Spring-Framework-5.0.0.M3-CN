### 18.9.3主题解析器

在定义主题之后，如上一节所述，决定使用哪个主题。 `DispatcherServlet`将查找一个名为`themeResolver`的bean来找出使用哪个`ThemeResolver`实现。 一个主题解析器的工作方式与`LocaleResolver`非常相似。 它检测用于特定请求的主题，也可以更改请求的主题。 Spring提供以下主题解析器：

**Table18.5.ThemeResolver实现**

| 类 | 描述 |
| :--- | :--- |
| `FixedThemeResolver` | 选择一个固定的主题，使用`defaultThemeName`属性设置。 |
| `SessionThemeResolver` | 主题维护在用户的HTTP会话中。它只需要为每个会话设置一次，但不会在会话之间持久化。 |
| `CookieThemeResolver` | 所选主题存储在客户端的cookie中。 |

Spring还提供了一个`ThemeChangeInterceptor`，它允许使用一个简单的请求参数来更改每个请求的主题。

