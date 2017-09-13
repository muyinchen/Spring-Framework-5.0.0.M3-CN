### 18.8.3CookieLocaleResolver

This locale resolver inspects a`Cookie`that might exist on the client to see if a`Locale`or`TimeZone`is specified. If so, it uses the specified details. Using the properties of this locale resolver, you can specify the name of the cookie as well as the maximum age. Find below an example of defining a`CookieLocaleResolver`.

```java
<bean id="localeResolver" class="org.springframework.web.servlet.i18n.CookieLocaleResolver">

	<property name="cookieName" value="clientlanguage"/>

	<!-- in seconds. If set to -1, the cookie is not persisted (deleted when browser shuts down) -->
	<property name="cookieMaxAge" value="100000"/>

</bean>
```

**Table18.4.CookieLocaleResolver properties**

| Property | Default | Description |
| :--- | :--- | :--- |
| cookieName | classname + LOCALE | The name of the cookie |
| cookieMaxAge | Servlet container default | The maximum time a cookie will stay persistent on the client. If -1 is specified, the cookie will not be persisted; it will only be available until the client shuts down their browser. |
| cookiePath | / | Limits the visibility of the cookie to a certain part of your site. When cookiePath is specified, the cookie will only be visible to that path and the paths below it. |



