### 18.8.5LocaleChangeInterceptor

You can enable changing of locales by adding the`LocaleChangeInterceptor`to one of the handler mappings \(see[Section18.4, “Handler mappings”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-handlermapping)\). It will detect a parameter in the request and change the locale. It calls`setLocale()`on the`LocaleResolver`that also exists in the context. The following example shows that calls to all`*.view`resources containing a parameter named`siteLanguage`will now change the locale. So, for example, a request for the following URL,`http://www.sf.net/home.view?siteLanguage=nl`will change the site language to Dutch.

您可以通过将`LocaleChangeInterceptor`添加到其中一个处理程序映射（请参见[第18.4节“处理程序映射”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-handlermapping)）来启用区域设置的更改。 它将检测请求中的参数并更改语言环境。 它调用上下文中也存在的`LocaleResolver`上的`setLocale()`。 以下示例显示调用包含名为`siteLanguage`的参数的所有`* .view`资源现在将更改语言环境。 因此，例如，对以下URL（`http://www.sf.net/home.view?siteLanguage=nl`）的请求会将网站语言更改为荷兰语。

```java
<bean id="localeChangeInterceptor"
        class="org.springframework.web.servlet.i18n.LocaleChangeInterceptor">
    <property name="paramName" value="siteLanguage"/>
</bean>

<bean id="localeResolver"
        class="org.springframework.web.servlet.i18n.CookieLocaleResolver"/>

<bean id="urlMapping"
        class="org.springframework.web.servlet.handler.SimpleUrlHandlerMapping">
    <property name="interceptors">
        <list>
            <ref bean="localeChangeInterceptor"/>
        </list>
    </property>
    <property name="mappings">
        <value>/**/*.view=someController</value>
    </property>
</bean>
```



