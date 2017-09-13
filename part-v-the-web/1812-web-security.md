## 18.12 Web Security

The[Spring Security](http://projects.spring.io/spring-security/)project provides features to protect web applications from malicious exploits. Check out the reference documentation in the sections on["CSRF protection"](http://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/#csrf),["Security Response Headers"](http://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/#headers), and also["Spring MVC Integration"](http://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/#mvc). Note that using Spring Security to secure the application is not necessarily required for all features. For example CSRF protection can be added simply by adding the`CsrfFilter`and`CsrfRequestDataValueProcessor`to your configuration. See the[Spring MVC Showcase](https://github.com/spring-projects/spring-mvc-showcase/commit/361adc124c05a8187b84f25e8a57550bb7d9f8e4)for an example.

Another option is to use a framework dedicated to Web Security.[HDIV](http://hdiv.org/)is one such framework and integrates with Spring MVC.

