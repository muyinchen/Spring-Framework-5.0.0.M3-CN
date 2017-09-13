### 18.14.4Shallow ETag support

Support for ETags is provided by the Servlet filter`ShallowEtagHeaderFilter`. It is a plain Servlet Filter, and thus can be used in combination with any web framework. The`ShallowEtagHeaderFilter`filter creates so-called shallow ETags \(as opposed to deep ETags, more about that later\).The filter caches the content of the rendered JSP \(or other content\), generates an MD5 hash over that, and returns that as an ETag header in the response. The next time a client sends a request for the same resource, it uses that hash as the`If-None-Match`value. The filter detects this, renders the view again, and compares the two hashes. If they are equal, a`304`is returned.

Note that this strategy saves network bandwidth but not CPU, as the full response must be computed for each request. Other strategies at the controller level \(described above\) can save network bandwidth and avoid computation.

This filter has a`writeWeakETag`parameter that configures the filter to write Weak ETags, like this:`W/"02a2d595e6ed9a0b24f027f2b63b134d6"`, as defined in[RFC 7232 Section 2.3](https://tools.ietf.org/html/rfc7232#section-2.3).

You configure the`ShallowEtagHeaderFilter`in`web.xml`:

```java
<filter>
	<filter-name>etagFilter</filter-name>
	<filter-class>org.springframework.web.filter.ShallowEtagHeaderFilter</filter-class>
	<!-- Optional parameter that configures the filter to write weak ETags
	<init-param>
       	<param-name>writeWeakETag</param-name>
       	<param-value>true</param-value>
	</init-param>
	-->
</filter>

<filter-mapping>
	<filter-name>etagFilter</filter-name>
	<servlet-name>petclinic</servlet-name>
</filter-mapping>
```

Or in Servlet 3.0+ environments,

```java
public class MyWebAppInitializer extends AbstractDispatcherServletInitializer {

	// ...

	@Override
	protected Filter[] getServletFilters() {
		return new Filter[] { new ShallowEtagHeaderFilter() };
	}

}
```

See[Section18.15, “Code-based Servlet container initialization”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-container-config)for more details.

