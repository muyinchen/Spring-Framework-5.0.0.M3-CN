### 11.6.2HtmlUnit Integration

Spring provides integration between[MockMvc](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/integration-testing.html#spring-mvc-test-server)and[HtmlUnit](http://htmlunit.sourceforge.net/). This simplifies performing end-to-end testing when using HTML based views. This integration enables developers to:

* Easily test HTML pages using tools such as[HtmlUnit](http://htmlunit.sourceforge.net/),[WebDriver](http://seleniumhq.org/projects/webdriver/), &[Geb](http://www.gebish.org/manual/current/testing.html#spock_junit__testng)without the need to deploy to a Servlet container

* Test JavaScript within pages

* Optionally test using mock services to speed up testing

* Share logic between in-container end-to-end tests and out-of-container integration tests

| ![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| :--- |
| `MockMvc`works with templating technologies that do not rely on a Servlet Container \(e.g., Thymeleaf, FreeMarker, etc.\), but it does not work with JSPs since they rely on the Servlet container. |

#### Why HtmlUnit Integration?

The most obvious question that comes to mind is, "Why do I need this?". The answer is best found by exploring a very basic sample application. Assume you have a Spring MVC web application that supports CRUD operations on a`Message`object. The application also supports paging through all messages. How would you go about testing it?

With Spring MVC Test, we can easily test if we are able to create a`Message`.

```java
MockHttpServletRequestBuilder createMessage = post("/messages/")
	.param("summary", "Spring Rocks")
	.param("text", "In case you didn't know, Spring Rocks!");

mockMvc.perform(createMessage)
	.andExpect(status().is3xxRedirection())
	.andExpect(redirectedUrl("/messages/123"));

```

What if we want to test our form view that allows us to create the message? For example, assume our form looks like the following snippet:

```java
<form id="messageForm" action="/messages/" method="post">
  <div class="pull-right"><a href="/messages/">Messages</a></div>

  <label for="summary">Summary</label>
  <input type="text" class="required" id="summary" name="summary" value="" />

  <label for="text">Message</label>
  <textarea id="text" name="text"></textarea>

  <div class="form-actions">
	<input type="submit" value="Create" />
  </div>
</form>
```

How do we ensure that our form will produce the correct request to create a new message? A naive attempt would look like this:

```java
mockMvc.perform(get("/messages/form"))
	.andExpect(xpath("//input[@name='summary']").exists())
	.andExpect(xpath("//textarea[@name='text']").exists());
```

This test has some obvious drawbacks. If we update our controller to use the parameter`message`instead of`text`, our form test would continue to pass even though the HTML form is out of synch with the controller. To resolve this we can combine our two tests.

```java
String summaryParamName = "summary";
String textParamName = "text";
mockMvc.perform(get("/messages/form"))
		.andExpect(xpath("//input[@name='" + summaryParamName + "']").exists())
		.andExpect(xpath("//textarea[@name='" + textParamName + "']").exists());

MockHttpServletRequestBuilder createMessage = post("/messages/")
		.param(summaryParamName, "Spring Rocks")
		.param(textParamName, "In case you didn't know, Spring Rocks!");

mockMvc.perform(createMessage)
		.andExpect(status().is3xxRedirection())
		.andExpect(redirectedUrl("/messages/123"));
```

This would reduce the risk of our test incorrectly passing, but there are still some problems.

* What if we have multiple forms on our page? Admittedly we could update our xpath expressions, but they get more complicated the more factors we take into account \(Are the fields the correct type? Are the fields enabled? etc.\).

* Another issue is that we are doing double the work we would expect. We must first verify the view, and then we submit the view with the same parameters we just verified. Ideally this could be done all at once.

* Finally, there are some things that we still cannot account for. For example, what if the form has JavaScript validation that we wish to test as well?

The overall problem is that testing a web page does not involve a single interaction. Instead, it is a combination of how the user interacts with a web page and how that web page interacts with other resources. For example, the result of a form view is used as the input to a user for creating a message. In addition, our form view may potentially utilize additional resources which impact the behavior of the page, such as JavaScript validation.

##### Integration testing to the rescue?

To resolve the issues above we could perform end-to-end integration testing, but this has some obvious drawbacks. Consider testing the view that allows us to page through the messages. We might need the following tests.

* Does our page display a notification to the user indicating that no results are available when the messages are empty?

* Does our page properly display a single message?

* Does our page properly support paging?

To set up these tests, we would need to ensure our database contained the proper messages in it. This leads to a number of additional challenges.

* Ensuring the proper messages are in the database can be tedious; consider foreign key constraints.

* Testing can become slow since each test would need to ensure that the database is in the correct state.

* Since our database needs to be in a specific state, we cannot run tests in parallel.

* Performing assertions on things like auto-generated ids, timestamps, etc. can be difficult.

These challenges do not mean that we should abandon end-to-end integration testing altogether. Instead, we can reduce the number of end-to-end integration tests by refactoring our detailed tests to use mock services which will execute much faster, more reliably, and without side effects. We can then implement a small number of_true_end-to-end integration tests that validate simple workflows to ensure that everything works together properly.

##### Enter HtmlUnit Integration

So how can we achieve a balance between testing the interactions of our pages and still retain good performance within our test suite? The answer is: "By integrating MockMvc with HtmlUnit."

##### HtmlUnit Integration Options

There are a number of ways to integrate`MockMvc`with HtmlUnit.

* [MockMvc and HtmlUnit](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/integration-testing.html#spring-mvc-test-server-htmlunit-mah): Use this option if you want to use the raw HtmlUnit libraries.

* [MockMvc and WebDriver](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/integration-testing.html#spring-mvc-test-server-htmlunit-webdriver): Use this option to ease development and reuse code between integration and end-to-end testing.

* [MockMvc and Geb](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/integration-testing.html#spring-mvc-test-server-htmlunit-geb): Use this option if you would like to use Groovy for testing, ease development, and reuse code between integration and end-to-end testing.

#### MockMvc and HtmlUnit

This section describes how to integrate`MockMvc`and HtmlUnit. Use this option if you want to use the raw HtmlUnit libraries.

##### MockMvc and HtmlUnit Setup

First, make sure that you have included a test dependency on`net.sourceforge.htmlunit:htmlunit`. In order to use HtmlUnit with Apache HttpComponents 4.5+, you will need to use HtmlUnit 2.18 or higher.

We can easily create an HtmlUnit`WebClient`that integrates with`MockMvc`using the`MockMvcWebClientBuilder`as follows.

```java

@Autowired
WebApplicationContext context;

WebClient webClient;

@Before
public void setup() {
	webClient = MockMvcWebClientBuilder
		.webAppContextSetup(context)
		.build();
}

```

| ![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| :--- |
| This is a simple example of using`MockMvcWebClientBuilder`. For advanced usage see[the section called “Advanced MockMvcWebClientBuilder”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/integration-testing.html#spring-mvc-test-server-htmlunit-mah-advanced-builder) |

This will ensure that any URL referencing`localhost`as the server will be directed to our`MockMvc`instance without the need for a real HTTP connection. Any other URL will be requested using a network connection as normal. This allows us to easily test the use of CDNs.

##### MockMvc and HtmlUnit Usage

Now we can use HtmlUnit as we normally would, but without the need to deploy our application to a Servlet container. For example, we can request the view to create a message with the following.

```java
HtmlPage createMsgFormPage = webClient.getPage("http://localhost/messages/form");
```

| ![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| :--- |
| The default context path is`""`. Alternatively, we can specify the context path as illustrated in[the section called “Advanced MockMvcWebClientBuilder”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/integration-testing.html#spring-mvc-test-server-htmlunit-mah-advanced-builder). |

Once we have a reference to the`HtmlPage`, we can then fill out the form and submit it to create a message.

```java
HtmlForm form = createMsgFormPage.getHtmlElementById("messageForm");
HtmlTextInput summaryInput = createMsgFormPage.getHtmlElementById("summary");
summaryInput.setValueAttribute("Spring Rocks");
HtmlTextArea textInput = createMsgFormPage.getHtmlElementById("text");
textInput.setText("In case you didn't know, Spring Rocks!");
HtmlSubmitInput submit = form.getOneHtmlElementByAttribute("input", "type", "submit");
HtmlPage newMessagePage = submit.click();
```

Finally, we can verify that a new message was created successfully. The following assertions use the[AssertJ](https://joel-costigliola.github.io/assertj/)library.

```java
assertThat(newMessagePage.getUrl().toString()).endsWith("/messages/123");
String id = newMessagePage.getHtmlElementById("id").getTextContent();
assertThat(id).isEqualTo("123");
String summary = newMessagePage.getHtmlElementById("summary").getTextContent();
assertThat(summary).isEqualTo("Spring Rocks");
String text = newMessagePage.getHtmlElementById("text").getTextContent();
assertThat(text).isEqualTo("In case you didn't know, Spring Rocks!");
```

This improves on our[MockMvc test](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/integration-testing.html#spring-mvc-test-server-htmlunit-mock-mvc-test)in a number of ways. First we no longer have to explicitly verify our form and then create a request that looks like the form. Instead, we request the form, fill it out, and submit it, thereby significantly reducing the overhead.

Another important factor is that[HtmlUnit uses the Mozilla Rhino engine](http://htmlunit.sourceforge.net/javascript.html)to evaluate JavaScript. This means that we can test the behavior of JavaScript within our pages as well!

Refer to the[HtmlUnit documentation](http://htmlunit.sourceforge.net/gettingStarted.html)for additional information about using HtmlUnit.

##### Advanced MockMvcWebClientBuilder

In the examples so far, we have used`MockMvcWebClientBuilder`in the simplest way possible, by building a`WebClient`based on the`WebApplicationContext`loaded for us by the Spring TestContext Framework. This approach is repeated here.

```java
@Autowired
WebApplicationContext context;

WebClient webClient;

@Before
public void setup() {
	webClient = MockMvcWebClientBuilder
		.webAppContextSetup(context)
		.build();
}
```

We can also specify additional configuration options.

```java
WebClient webClient;

@Before
public void setup() {
	webClient = MockMvcWebClientBuilder
		// demonstrates applying a MockMvcConfigurer (Spring Security)
		.webAppContextSetup(context, springSecurity())
		// for illustration only - defaults to ""
		.contextPath("")
		// By default MockMvc is used for localhost only;
		// the following will use MockMvc for example.com and example.org as well
		.useMockMvcForHosts("example.com","example.org")
		.build();
}
```

As an alternative, we can perform the exact same setup by configuring the`MockMvc`instance separately and supplying it to the`MockMvcWebClientBuilder`as follows.

```java
MockMvc mockMvc = MockMvcBuilders
		.webAppContextSetup(context)
		.apply(springSecurity())
		.build();

webClient = MockMvcWebClientBuilder
		.mockMvcSetup(mockMvc)
		// for illustration only - defaults to ""
		.contextPath("")
		// By default MockMvc is used for localhost only;
		// the following will use MockMvc for example.com and example.org as well
		.useMockMvcForHosts("example.com","example.org")
		.build();
```

This is more verbose, but by building the`WebClient`with a`MockMvc`instance we have the full power of`MockMvc`at our fingertips.

|  |
| :--- |
| For additional information on creating a`MockMvc`instance refer to[the section called “Setup Choices”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/integration-testing.html#spring-mvc-test-server-setup-options). |

#### MockMvc and WebDriver

In the previous sections, we have seen how to use`MockMvc`in conjunction with the raw HtmlUnit APIs. In this section, we will leverage additional abstractions within the Selenium[WebDriver](http://docs.seleniumhq.org/projects/webdriver/)to make things even easier.

##### Why WebDriver and MockMvc?

We can already use HtmlUnit and`MockMvc`, so why would we want to use`WebDriver`? The Selenium`WebDriver`provides a very elegant API that allows us to easily organize our code. To better understand, let’s explore an example.

| ![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| :--- |
| Despite being a part of[Selenium](http://docs.seleniumhq.org/), WebDriver does not require a Selenium Server to run your tests. |

Suppose we need to ensure that a message is created properly. The tests involve finding the HTML form input elements, filling them out, and making various assertions.

This approach results in numerous, separate tests because we want to test error conditions as well. For example, we want to ensure that we get an error if we fill out only part of the form. If we fill out the entire form, the newly created message should be displayed afterwards.

If one of the fields were named "summary", then we might have something like the following repeated in multiple places within our tests.

```java
HtmlTextInput summaryInput = currentPage.getHtmlElementById("summary");
summaryInput.setValueAttribute(summary);
```

So what happens if we change the`id`to "smmry"? Doing so would force us to update all of our tests to incorporate this change! Of course, this violates the_DRY Principle_; so we should ideally extract this code into its own method as follows.

```java
public HtmlPage createMessage(HtmlPage currentPage, String summary, String text) {
	setSummary(currentPage, summary);
	// ...
}

public void setSummary(HtmlPage currentPage, String summary) {
	HtmlTextInput summaryInput = currentPage.getHtmlElementById("summary");
	summaryInput.setValueAttribute(summary);
}
```

This ensures that we do not have to update all of our tests if we change the UI.

We might even take this a step further and place this logic within an Object that represents the`HtmlPage`we are currently on.

```java
public class CreateMessagePage {

	final HtmlPage currentPage;

	final HtmlTextInput summaryInput;

	final HtmlSubmitInput submit;

	public CreateMessagePage(HtmlPage currentPage) {
		this.currentPage = currentPage;
		this.summaryInput = currentPage.getHtmlElementById("summary");
		this.submit = currentPage.getHtmlElementById("submit");
	}

	public <T> T createMessage(String summary, String text) throws Exception {
		setSummary(summary);

		HtmlPage result = submit.click();
		boolean error = CreateMessagePage.at(result);

		return (T) (error ? new CreateMessagePage(result) : new ViewMessagePage(result));
	}

	public void setSummary(String summary) throws Exception {
		summaryInput.setValueAttribute(summary);
	}

	public static boolean at(HtmlPage page) {
		return "Create Message".equals(page.getTitleText());
	}
}
```

Formerly, this pattern is known as the[Page Object Pattern](https://code.google.com/p/selenium/wiki/PageObjects). While we can certainly do this with HtmlUnit, WebDriver provides some tools that we will explore in the following sections to make this pattern much easier to implement.

##### MockMvc and WebDriver Setup

To use Selenium WebDriver with the Spring MVC Test framework, make sure that your project includes a test dependency on`org.seleniumhq.selenium:selenium-htmlunit-driver`.

We can easily create a Selenium`WebDriver`that integrates with`MockMvc`using the`MockMvcHtmlUnitDriverBuilder`as follows.

```java
@Autowired
WebApplicationContext context;

WebDriver driver;

@Before
public void setup() {
	driver = MockMvcHtmlUnitDriverBuilder
		.webAppContextSetup(context)
		.build();
}
```

| ![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png.pagespeed.ce.9zQ_1wVwzR.png "\[Note\]") |
| :--- |
| This is a simple example of using`MockMvcHtmlUnitDriverBuilder`. For more advanced usage, refer to[the section called “Advanced MockMvcHtmlUnitDriverBuilder”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/integration-testing.html#spring-mvc-test-server-htmlunit-webdriver-advanced-builder) |

This will ensure that any URL referencing`localhost`as the server will be directed to our`MockMvc`instance without the need for a real HTTP connection. Any other URL will be requested using a network connection as normal. This allows us to easily test the use of CDNs.

##### MockMvc and WebDriver Usage

Now we can use WebDriver as we normally would, but without the need to deploy our application to a Servlet container. For example, we can request the view to create a message with the following.

```java
CreateMessagePage page = CreateMessagePage.to(driver);
```

We can then fill out the form and submit it to create a message.

```java
ViewMessagePage viewMessagePage =
	page.createMessage(ViewMessagePage.class, expectedSummary, expectedText);
```

This improves on the design of our[HtmlUnit test](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/integration-testing.html#spring-mvc-test-server-htmlunit-mah-usage)by leveraging the_Page Object Pattern_. As we mentioned in[the section called “Why WebDriver and MockMvc?”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/integration-testing.html#spring-mvc-test-server-htmlunit-webdriver-why), we can use the Page Object Pattern with HtmlUnit, but it is much easier with WebDriver. Let’s take a look at our new`CreateMessagePage`implementation.

```java
public class CreateMessagePage extends AbstractPage { 

	
	private WebElement summary;
	private WebElement text;
	
	
	@FindBy(css = "input[type=submit]")
	private WebElement submit;
	
	public CreateMessagePage(WebDriver driver) {
		super(driver);
	}
	
	public <T> T createMessage(Class<T> resultPage, String summary, String details) {
		this.summary.sendKeys(summary);
		this.text.sendKeys(details);
		this.submit.click();
		return PageFactory.initElements(driver, resultPage);
	}
	
	public static CreateMessagePage to(WebDriver driver) {
		driver.get("http://localhost:9990/mail/messages/form");
		return PageFactory.initElements(driver, CreateMessagePage.class);
	}
}
```

| [![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/callouts/1.png "1")](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/integration-testing.html#CO1-1) |
| :--- |


|  | The first thing you will notice is that`CreateMessagePage`extends the`AbstractPage`. We won’t go over the details of`AbstractPage`, but in summary it contains common functionality for all of our pages. For example, if our application has a navigational bar, global error messages, etc., this logic can be placed in a shared location. |
| :--- | :--- |
| [![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/callouts/2.png "2")](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/integration-testing.html#CO1-2) | The next thing you will notice is that we have a member variable for each of the parts of the HTML page that we are interested in. These are of type`WebElement`.`WebDriver`'s[PageFactory](https://code.google.com/p/selenium/wiki/PageFactory)allows us to remove a lot of code from the HtmlUnit version of`CreateMessagePage`by automatically resolving each`WebElement`. The[PageFactory\#initElements\(WebDriver,Class&lt;T&gt;\)](https://selenium.googlecode.com/git/docs/api/java/org/openqa/selenium/support/PageFactory.html#initElements-org.openqa.selenium.WebDriver-java.lang.Class-)method will automatically resolve each`WebElement`by using the field name and looking it up by the`id`or`name`of the element within the HTML page. |
| [![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/callouts/3.png "3")](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/integration-testing.html#CO1-3) | We can use the[@FindBy annotation](https://code.google.com/p/selenium/wiki/PageFactory#Making_the_Example_Work_Using_Annotations)to override the default lookup behavior. Our example demonstrates how to use the`@FindBy`annotation to look up our submit button using a css selector,**input\[type=submit\]**. |

Finally, we can verify that a new message was created successfully. The following assertions use the[FEST assertion library](https://code.google.com/p/fest/).

```java
assertThat(viewMessagePage.getMessage()).isEqualTo(expectedMessage);
assertThat(viewMessagePage.getSuccess()).isEqualTo("Successfully created a new message");
```

We can see that our`ViewMessagePage`allows us to interact with our custom domain model. For example, it exposes a method that returns a`Message`object.

```java
public Message getMessage() throws ParseException {
	Message message = new Message();
	message.setId(getId());
	message.setCreated(getCreated());
	message.setSummary(getSummary());
	message.setText(getText());
	return message;
}
```

We can then leverage the rich domain objects in our assertions.

Lastly, don’t forget to_close_the`WebDriver`instance when the test is complete.

```
@After
public void destroy() {
	if (driver != null) {
		driver.close();
	}
}
```

For additional information on using WebDriver, refer to the Selenium[WebDriver documentation](https://code.google.com/p/selenium/wiki/GettingStarted).

##### Advanced MockMvcHtmlUnitDriverBuilder

In the examples so far, we have used`MockMvcHtmlUnitDriverBuilder`in the simplest way possible, by building a`WebDriver`based on the`WebApplicationContext`loaded for us by the Spring TestContext Framework. This approach is repeated here.

```java
@Autowired
WebApplicationContext context;

WebDriver driver;

@Before
public void setup() {
	driver = MockMvcHtmlUnitDriverBuilder
		.webAppContextSetup(context)
		.build();
}
```

We can also specify additional configuration options.

```java
WebDriver driver;

@Before
public void setup() {
	driver = MockMvcHtmlUnitDriverBuilder
		// demonstrates applying a MockMvcConfigurer (Spring Security)
		.webAppContextSetup(context, springSecurity())
		// for illustration only - defaults to ""
		.contextPath("")
		// By default MockMvc is used for localhost only;
		// the following will use MockMvc for example.com and example.org as well
		.useMockMvcForHosts("example.com","example.org")
		.build();
}
```

As an alternative, we can perform the exact same setup by configuring the`MockMvc`instance separately and supplying it to the`MockMvcHtmlUnitDriverBuilder`as follows.

```java
MockMvc mockMvc = MockMvcBuilders
		.webAppContextSetup(context)
		.apply(springSecurity())
		.build();

driver = MockMvcHtmlUnitDriverBuilder
		.mockMvcSetup(mockMvc)
		// for illustration only - defaults to ""
		.contextPath("")
		// By default MockMvc is used for localhost only;
		// the following will use MockMvc for example.com and example.org as well
		.useMockMvcForHosts("example.com","example.org")
		.build();
```

This is more verbose, but by building the`WebDriver`with a`MockMvc`instance we have the full power of`MockMvc`at our fingertips.

| ![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/tip.png.pagespeed.ce.w22Wv-tZ37.png "\[Tip\]") |
| :--- |
| For additional information on creating a`MockMvc`instance refer to[the section called “Setup Choices”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/integration-testing.html#spring-mvc-test-server-setup-options). |

#### MockMvc and Geb

In the previous section, we saw how to use`MockMvc`with`WebDriver`. In this section, we will use[Geb](http://www.gebish.org/)to make our tests even Groovy-er.

##### Why Geb and MockMvc?

Geb is backed by WebDriver, so it offers many of the[same benefits](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/integration-testing.html#spring-mvc-test-server-htmlunit-webdriver-why)that we get from WebDriver. However, Geb makes things even easier by taking care of some of the boilerplate code for us.

##### MockMvc and Geb Setup

We can easily initialize a Geb`Browser`with a Selenium`WebDriver`that uses`MockMvc`as follows.

```java
def setup() {
	browser.driver = MockMvcHtmlUnitDriverBuilder
		.webAppContextSetup(context)
		.build()
}
```

| ![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png.pagespeed.ce.9zQ_1wVwzR.png "\[Note\]") |
| :--- |
| This is a simple example of using`MockMvcHtmlUnitDriverBuilder`. For more advanced usage, refer to[the section called “Advanced MockMvcHtmlUnitDriverBuilder”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/integration-testing.html#spring-mvc-test-server-htmlunit-webdriver-advanced-builder) |

This will ensure that any URL referencing`localhost`as the server will be directed to our`MockMvc`instance without the need for a real HTTP connection. Any other URL will be requested using a network connection as normal. This allows us to easily test the use of CDNs.

##### MockMvc and Geb Usage

Now we can use Geb as we normally would, but without the need to deploy our application to a Servlet container. For example, we can request the view to create a message with the following:

```java
to CreateMessagePage
```

We can then fill out the form and submit it to create a message.

```java
when:
form.summary = expectedSummary
form.text = expectedMessage
submit.click(ViewMessagePage)
```

Any unrecognized method calls or property accesses/references that are not found will be forwarded to the current page object. This removes a lot of the boilerplate code we needed when using WebDriver directly.

As with direct WebDriver usage, this improves on the design of our[HtmlUnit test](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/integration-testing.html#spring-mvc-test-server-htmlunit-mah-usage)by leveraging the_Page Object Pattern_. As mentioned previously, we can use the Page Object Pattern with HtmlUnit and WebDriver, but it is even easier with Geb. Let’s take a look at our new Groovy-based`CreateMessagePage`implementation.

```java
class CreateMessagePage extends Page {
	static url = 'messages/form'
	static at = { assert title == 'Messages : Create'; true }
	static content =  {
		submit { $('input[type=submit]') }
		form { $('form') }
		errors(required:false) { $('label.error, .alert-error')?.text() }
	}
}
```

The first thing you will notice is that our`CreateMessagePage`extends`Page`. We won’t go over the details of`Page`, but in summary it contains common functionality for all of our pages. The next thing you will notice is that we define a URL in which this page can be found. This allows us to navigate to the page as follows.

```java
to CreateMessagePage
```

We also have an`at`closure that determines if we are at the specified page. It should return`true`if we are on the correct page. This is why we can assert that we are on the correct page as follows.

```java
then:
at CreateMessagePage
errors.contains(
'This field is required.'
)
```

| ![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png.pagespeed.ce.9zQ_1wVwzR.png "\[Note\]") |
| :--- |
| We use an assertion in the closure, so that we can determine where things went wrong if we were at the wrong page. |

Next we create a`content`closure that specifies all the areas of interest within the page. We can use a[jQuery-ish Navigator API](http://www.gebish.org/manual/current/intro.html#the_jquery_ish_navigator_api)to select the content we are interested in.

Finally, we can verify that a new message was created successfully.

```java
then:
at ViewMessagePage
success == 
'Successfully created a new message'

id
date
summary == expectedSummary
message == expectedMessage
```

For further details on how to get the most out of Geb, consult[The Book of Geb](http://www.gebish.org/manual/current/)user’s manual.

