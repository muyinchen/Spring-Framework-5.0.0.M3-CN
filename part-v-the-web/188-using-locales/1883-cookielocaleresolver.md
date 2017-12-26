### 18.8.3CookieLocaleResolver

此语言环境解析程序将检查客户端上可能存在的`Cookie`，以查看是否指定了`Locale`或`TimeZone`。 如果是这样，它使用指定的细节。 使用此语言环境解析器的属性，您可以指定cookie的名称以及最大年龄。 下面找到一个定义`CookieLocaleResolver`的例子。

```java
<bean id="localeResolver" class="org.springframework.web.servlet.i18n.CookieLocaleResolver">

    <property name="cookieName" value="clientlanguage"/>

    <!-- 以秒为单位。如果设置为-1，则cookie不会持久化（在浏览器关闭时删除） -->
    <property name="cookieMaxAge" value="100000"/>

</bean>
```

**Table18.4.CookieLocaleResolver 属性**

| 属性 | 默认 | 描述 |
| :--- | :--- | :--- |
| cookieName | classname + LOCALE | cookie的名称 |
| cookieMaxAge | Servlet容器默认 | Cookie在客户端上保持持续的最长时间。如果指定了-1，则cookie不会被持久化; 只有客户端关闭浏览器才可用。 |
| cookiePath | / | 将Cookie的可见性限制在您网站的某个部分。指定cookiePath时，cookie将只对该路径及其下方的路径可见。. |



