## 19.11Feed Views

Both`AbstractAtomFeedView`and`AbstractRssFeedView`inherit from the base class`AbstractFeedView`and are used to provide Atom and RSS Feed views respectfully. They are based on java.net’s[ROME](https://rome.dev.java.net/)project and are located in the package`org.springframework.web.servlet.view.feed`.

`AbstractAtomFeedView`requires you to implement the`buildFeedEntries()`method and optionally override the`buildFeedMetadata()`method \(the default implementation is empty\), as shown below.

```java
public class SampleContentAtomView extends AbstractAtomFeedView {

	@Override
	protected void buildFeedMetadata(Map<String, Object> model,
			Feed feed, HttpServletRequest request) {
		// implementation omitted
	}

	@Override
	protected List<Entry> buildFeedEntries(Map<String, Object> model,
			HttpServletRequest request, HttpServletResponse response) throws Exception {
		// implementation omitted
	}

}
```

Similar requirements apply for implementing`AbstractRssFeedView`, as shown below.

```java
public class SampleContentAtomView extends AbstractRssFeedView {

	@Override
	protected void buildFeedMetadata(Map<String, Object> model,
			Channel feed, HttpServletRequest request) {
		// implementation omitted
	}

	@Override
	protected List<Item> buildFeedItems(Map<String, Object> model,
			HttpServletRequest request, HttpServletResponse response) throws Exception {
		// implementation omitted
	}

}
```

The`buildFeedItems()`and`buildFeedEntires()`methods pass in the HTTP request in case you need to access the Locale. The HTTP response is passed in only for the setting of cookies or other HTTP headers. The feed will automatically be written to the response object after the method returns.

For an example of creating an Atom view please refer to Alef Arendsen’s Spring Team Blog[entry](https://spring.io/blog/2009/03/16/adding-an-atom-view-to-an-application-using-spring-s-rest-support).

