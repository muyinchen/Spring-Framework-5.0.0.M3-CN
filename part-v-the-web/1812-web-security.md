## 18.12 网络安全

[Spring Security](http://projects.spring.io/spring-security/)项目提供了保护Web应用程序免受恶意攻击的功能。 查看[“CSRF保护”](http://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/#csrf)，[“安全响应头”](http://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/#headers)以及[“Spring MVC集成”部分中的参考文档](http://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/#mvc)。 请注意，使用Spring Security来保护应用程序不一定是所有功能都需要的。 例如，可以通过将`CsrfFilter`和`CsrfRequestDataValueProcessor`添加到您的配置来添加CSRF保护。 请参阅[Spring MVC展示](https://github.com/spring-projects/spring-mvc-showcase/commit/361adc124c05a8187b84f25e8a57550bb7d9f8e4)示例。

另一个选择是使用专用于Web Security的框架。[HDIV](http://hdiv.org/)就是一个这样的框架，并且与Spring MVC集成在一起。

