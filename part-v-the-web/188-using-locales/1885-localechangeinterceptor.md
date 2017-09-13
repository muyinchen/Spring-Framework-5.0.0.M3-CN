### 18.8.5LocaleChangeInterceptor

You can enable changing of locales by adding the`LocaleChangeInterceptor`to one of the handler mappings \(see[Section18.4, “Handler mappings”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-handlermapping)\). It will detect a parameter in the request and change the locale. It calls`setLocale()`on the`LocaleResolver`that also exists in the context. The following example shows that calls to all`*.view`resources containing a parameter named`siteLanguage`will now change the locale. So, for example, a request for the following URL,`http://www.sf.net/home.view?siteLanguage=nl`will change the site language to Dutch.

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

  


