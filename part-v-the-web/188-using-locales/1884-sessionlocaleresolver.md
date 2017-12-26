### 18.8.4 SessionLocaleResolver

`SessionLocaleResolver`允许您从可能与用户请求关联的会话中检索`Locale`和`TimeZone`。 与`CookieLocaleResolver`相比，这种策略将本地选择的区域设置存储在Servlet容器的`HttpSession`中。 因此，这些设置对于每个会话都是临时的，因此在每个会话终止时都会丢失。

请注意，与Spring Session项目等外部会话管理机制没有直接关系。 这个`SessionLocaleResolver`将简单地根据当前的`HttpServletRequest`评估和修改相应的`HttpSession`属性。

