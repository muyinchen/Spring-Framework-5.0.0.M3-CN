### 11.6.1 Server-Side Tests

It’s easy to write a plain unit test for a Spring MVC controller using JUnit or TestNG: simply instantiate the controller, inject it with mocked or stubbed dependencies, and call its methods passing`MockHttpServletRequest`,`MockHttpServletResponse`, etc., as necessary. However, when writing such a unit test, much remains untested: for example, request mappings, data binding, type conversion, validation, and much more. Furthermore, other controller methods such as`@InitBinder`,`@ModelAttribute`, and`@ExceptionHandler`may also be invoked as part of the request processing lifecycle.

The goal of_Spring MVC Test_is to provide an effective way for testing controllers by performing requests and generating responses through the actual`DispatcherServlet`.

_Spring MVC Test_builds on the familiar["mock" implementations of the Servlet API](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/unit-testing.html#mock-objects-servlet)available in the`spring-test`module. This allows performing requests and generating responses without the need for running in a Servlet container. For the most part everything should work as it does at runtime with a few notable exceptions as explained in[the section called “Differences between Out-of-Container and End-to-End Integration Tests”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/integration-testing.html#spring-mvc-test-vs-end-to-end-integration-tests). Here is a JUnit 4 based example of using Spring MVC Test:

```
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;

import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;


@RunWith(SpringRunner.class)
@WebAppConfiguration
@ContextConfiguration("test-servlet-context.xml")
public
class
 ExampleTests {

	
@Autowired
private
 WebApplicationContext wac;

	
private
 MockMvc mockMvc;

	
@Before
public
void
 setup() {
		
this
.mockMvc = MockMvcBuilders.webAppContextSetup(
this
.wac).build();
	}

	
@Test
public
void
 getAccount() 
throws
 Exception {
		
this
.mockMvc.perform(get(
"/accounts/1"
).accept(MediaType.parseMediaType(
"application/json;charset=UTF-8"
)))
			.andExpect(status().isOk())
			.andExpect(content().contentType(
"application/json"
))
			.andExpect(jsonPath(
"$.name"
).value(
"Lee"
));
	}

}
```

The above test relies on the`WebApplicationContext`support of the_TestContext framework_for loading Spring configuration from an XML configuration file located in the same package as the test class, but Java-based and Groovy-based configuration are also supported. See these[sample tests](https://github.com/spring-projects/spring-framework/tree/master/spring-test/src/test/java/org/springframework/test/web/servlet/samples/context).

The`MockMvc`instance is used to perform a`GET`request to`"/accounts/1"`and verify that the resulting response has status 200, the content type is`"application/json"`, and the response body has a JSON property called "name" with the value "Lee". The`jsonPath`syntax is supported through the Jayway[JsonPath project](https://github.com/jayway/JsonPath). There are lots of other options for verifying the result of the performed request that will be discussed below.

#### Static Imports

The fluent API in the example above requires a few static imports such as`MockMvcRequestBuilders.*`,`MockMvcResultMatchers.*`, and`MockMvcBuilders.*`. An easy way to find these classes is to search for types matching_"MockMvc\*"_. If using Eclipse, be sure to add them as "favorite static members" in the Eclipse preferences under_Java → Editor → Content Assist → Favorites_. That will allow use of content assist after typing the first character of the static method name. Other IDEs \(e.g. IntelliJ\) may not require any additional configuration. Just check the support for code completion on static members.

#### Setup Choices

There are two main options for creating an instance of`MockMvc`. The first is to load Spring MVC configuration through the_TestContext framework_, which loads the Spring configuration and injects a`WebApplicationContext`into the test to use to build a`MockMvc`instance:

```
@RunWith(SpringRunner.class)
@WebAppConfiguration
@ContextConfiguration("my-servlet-context.xml")
public
class
 MyWebTests {

	
@Autowired
private
 WebApplicationContext wac;

	
private
 MockMvc mockMvc;

	
@Before
public
void
 setup() {
		
this
.mockMvc = MockMvcBuilders.webAppContextSetup(
this
.wac).build();
	}

	
// ...


}
```

The second is to simply create a controller instance manually without loading Spring configuration. Instead basic default configuration, roughly comparable to that of the MVC JavaConfig or the MVC namespace, is automatically created and can be customized to a degree:

```
public
class
 MyWebTests {

	
private
 MockMvc mockMvc;

	
@Before
public
void
 setup() {
		
this
.mockMvc = MockMvcBuilders.standaloneSetup(
new
 AccountController()).build();
	}

	
// ...


}
```

Which setup option should you use?

The_"webAppContextSetup"_loads your actual Spring MVC configuration resulting in a more complete integration test. Since the_TestContext framework_caches the loaded Spring configuration, it helps keep tests running fast, even as you introduce more tests in your test suite. Furthermore, you can inject mock services into controllers through Spring configuration in order to remain focused on testing the web layer. Here is an example of declaring a mock service with Mockito:

```
<
bean
id
=
"accountService"
class
=
"org.mockito.Mockito"
factory-method
=
"mock"
>
<
constructor-arg
value
=
"org.example.AccountService"
/
>
<
/bean
>
```

You can then inject the mock service into the test in order set up and verify expectations:

```
@RunWith(SpringRunner.class)
@WebAppConfiguration
@ContextConfiguration("test-servlet-context.xml")
public
class
 AccountTests {

	
@Autowired
private
 WebApplicationContext wac;

	
private
 MockMvc mockMvc;

	
@Autowired
private
 AccountService accountService;

	
// ...


}
```

The_"standaloneSetup"_on the other hand is a little closer to a unit test. It tests one controller at a time: the controller can be injected with mock dependencies manually, and it doesn’t involve loading Spring configuration. Such tests are more focused on style and make it easier to see which controller is being tested, whether any specific Spring MVC configuration is required to work, and so on. The "standaloneSetup" is also a very convenient way to write ad-hoc tests to verify specific behavior or to debug an issue.

Just like with any "integration vs. unit testing" debate, there is no right or wrong answer. However, using the "standaloneSetup" does imply the need for additional "webAppContextSetup" tests in order to verify your Spring MVC configuration. Alternatively, you may choose to write all tests with "webAppContextSetup" in order to always test against your actual Spring MVC configuration.

#### Setup Features

No matter which MockMvc builder you use all`MockMvcBuilder`implementations provide some common and very useful features. For example you can declare an`Accept`header for all requests and expect a status of 200 as well as a`Content-Type`header in all responses as follows:

```
// static import of MockMvcBuilders.standaloneSetup


MockMVc mockMvc = standaloneSetup(
new
 MusicController())
		.defaultRequest(get(
"/"
).accept(MediaType.APPLICATION_JSON))
		.alwaysExpect(status().isOk())
		.alwaysExpect(content().contentType(
"application/json;charset=UTF-8"
))
		.build();
```

In addition 3rd party frameworks \(and applications\) may pre-package setup instructions like the ones through a`MockMvcConfigurer`. The Spring Framework has one such built-in implementation that helps to save and re-use the HTTP session across requests. It can be used as follows:

```
// static import of SharedHttpSessionConfigurer.sharedHttpSession


MockMvc mockMvc = MockMvcBuilders.standaloneSetup(
new
 TestController())
        .apply(sharedHttpSession())
        .build();


// Use mockMvc to perform requests...
```

See`ConfigurableMockMvcBuilder`for a list of all MockMvc builder features or use the IDE to explore the available options.

#### Performing Requests

It’s easy to perform requests using any HTTP method:

```
mockMvc.perform(post(
"/hotels/{id}"
, 
42
).accept(MediaType.APPLICATION_JSON));
```

You can also perform file upload requests that internally use`MockMultipartHttpServletRequest`so that there is no actual parsing of a multipart request but rather you have to set it up:

```
mockMvc.perform(multipart(
"/doc"
).file(
"a1"
, 
"ABC"
.getBytes(
"UTF-8"
)));
```

You can specify query parameters in URI template style:

```
mockMvc.perform(get(
"/hotels?foo={foo}"
, 
"bar"
));
```

Or you can add Servlet request parameters representing either query of form parameters:

```
mockMvc.perform(get(
"/hotels"
).param(
"foo"
, 
"bar"
));
```

If application code relies on Servlet request parameters and doesn’t check the query string explicitly \(as is most often the case\) then it doesn’t matter which option you use. Keep in mind however that query params provided with the URI template will be decoded while request parameters provided through the`param(…​)`method are expected to already be decoded.

In most cases it’s preferable to leave out the context path and the Servlet path from the request URI. If you must test with the full request URI, be sure to set the`contextPath`and`servletPath`accordingly so that request mappings will work:

```
mockMvc.perform(get(
"/app/main/hotels/{id}"
).contextPath(
"/app"
).servletPath(
"/main"
))
```

Looking at the above example, it would be cumbersome to set the contextPath and servletPath with every performed request. Instead you can set up default request properties:

```
public
class
 MyWebTests {

	
private
 MockMvc mockMvc;

	
@Before
public
void
 setup() {
		mockMvc = standaloneSetup(
new
 AccountController())
			.defaultRequest(get(
"/"
)
			.contextPath(
"/app"
).servletPath(
"/main"
)
			.accept(MediaType.APPLICATION_JSON).build();
	}
```

The above properties will affect every request performed through the`MockMvc`instance. If the same property is also specified on a given request, it overrides the default value. That is why the HTTP method and URI in the default request don’t matter since they must be specified on every request.

#### Defining Expectations

Expectations can be defined by appending one or more`.andExpect(..)`calls after performing a request:

```
mockMvc.perform(get(
"/accounts/1"
)).andExpect(status().isOk());
```

`MockMvcResultMatchers.*`provides a number of expectations, some of which are further nested with more detailed expectations.

Expectations fall in two general categories. The first category of assertions verifies properties of the response: for example, the response status, headers, and content. These are the most important results to assert.

The second category of assertions goes beyond the response. These assertions allow one to inspect Spring MVC specific aspects such as which controller method processed the request, whether an exception was raised and handled, what the content of the model is, what view was selected, what flash attributes were added, and so on. They also allow one to inspect Servlet specific aspects such as request and session attributes.

The following test asserts that binding or validation failed:

```
mockMvc.perform(post(
"/persons"
))
	.andExpect(status().isOk())
	.andExpect(model().attributeHasErrors(
"person"
));
```

Many times when writing tests, it’s useful to_dump_the results of the performed request. This can be done as follows, where`print()`is a static import from`MockMvcResultHandlers`:

```
mockMvc.perform(post(
"/persons"
))
	.andDo(print())
	.andExpect(status().isOk())
	.andExpect(model().attributeHasErrors(
"person"
));
```

As long as request processing does not cause an unhandled exception, the`print()`method will print all the available result data to`System.out`. Spring Framework 4.2 introduced a`log()`method and two additional variants of the`print()`method, one that accepts an`OutputStream`and one that accepts a`Writer`. For example, invoking`print(System.err)`will print the result data to`System.err`; while invoking`print(myWriter)`will print the result data to a custom writer. If you would like to have the result data_logged_instead of printed, simply invoke the`log()`method which will log the result data as a single`DEBUG`message under the`org.springframework.test.web.servlet.result`logging category.

In some cases, you may want to get direct access to the result and verify something that cannot be verified otherwise. This can be achieved by appending`.andReturn()`after all other expectations:

```
MvcResult mvcResult = mockMvc.perform(post(
"/persons"
)).andExpect(status().isOk()).andReturn();

// ...
```

If all tests repeat the same expectations you can set up common expectations once when building the`MockMvc`instance:

```
standaloneSetup(
new
 SimpleController())
	.alwaysExpect(status().isOk())
	.alwaysExpect(content().contentType(
"application/json;charset=UTF-8"
))
	.build()
```

Note that common expectations are_always_applied and cannot be overridden without creating a separate`MockMvc`instance.

When JSON response content contains hypermedia links created with[Spring HATEOAS](https://github.com/spring-projects/spring-hateoas), the resulting links can be verified using JsonPath expressions:

```
mockMvc.perform(get(
"/people"
).accept(MediaType.APPLICATION_JSON))
	.andExpect(jsonPath(
"$.links[?(@.rel == 'self')].href"
).value(
"http://localhost:8080/people"
));
```

When XML response content contains hypermedia links created with[Spring HATEOAS](https://github.com/spring-projects/spring-hateoas), the resulting links can be verified using XPath expressions:

```
Map
<
String, String
>
 ns = Collections.singletonMap(
"ns"
, 
"http://www.w3.org/2005/Atom"
);
mockMvc.perform(get(
"/handle"
).accept(MediaType.APPLICATION_XML))
	.andExpect(xpath(
"/person
s:link[@rel='self']/@href"
, ns).string(
"http://localhost:8080/people"
));
```

#### Filter Registrations

When setting up a`MockMvc`instance, you can register one or more Servlet`Filter`instances:

```
mockMvc = standaloneSetup(
new
 PersonController()).addFilters(
new
 CharacterEncodingFilter()).build();
```

Registered filters will be invoked through via the`MockFilterChain`from`spring-test`, and the last filter will delegate to the`DispatcherServlet`.

#### Differences between Out-of-Container and End-to-End Integration Tests

As mentioned earlier_Spring MVC Test_is built on the Servlet API mock objects from the`spring-test`module and does not use a running Servlet container. Therefore there are some important differences compared to full end-to-end integration tests with an actual client and server running.

The easiest way to think about this is starting with a blank`MockHttpServletRequest`. Whatever you add to it is what the request will be. Things that may catch you by surprise are that there is no context path by default, no`jsessionid`cookie, no forwarding, error, or async dispatches, and therefore no actual JSP rendering. Instead, "forwarded" and "redirected" URLs are saved in the`MockHttpServletResponse`and can be asserted with expectations.

This means if you are using JSPs you can verify the JSP page to which the request was forwarded, but there won’t be any HTML rendered. In other words, the JSP will not be_invoked_. Note however that all other rendering technologies which don’t rely on forwarding such as Thymeleaf and Freemarker will render HTML to the response body as expected. The same is true for rendering JSON, XML, and other formats via`@ResponseBody`methods.

Alternatively you may consider the full end-to-end integration testing support from Spring Boot via`@WebIntegrationTest`. See the[Spring Boot reference](http://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-testing.html#boot-features-testing-spring-boot-applications).

There are pros and cons for each approach. The options provided in_Spring MVC Test_are different stops on the scale from classic unit testing to full integration testing. To be certain, none of the options in Spring MVC Test fall under the category of classic unit testing, but they_are_a little closer to it. For example, you can isolate the web layer by injecting mocked services into controllers, in which case you’re testing the web layer only through the`DispatcherServlet`but with actual Spring configuration, just like you might test the data access layer in isolation from the layers above. Or you can use the standalone setup focusing on one controller at a time and manually providing the configuration required to make it work.

Another important distinction when using_Spring MVC Test_is that conceptually such tests are on the_inside_of the server-side so you can check what handler was used, if an exception was handled with a HandlerExceptionResolver, what the content of the model is, what binding errors there were, etc. That means it’s easier to write expectations since the server is not a black box as it is when testing it through an actual HTTP client. This is generally an advantage of classic unit testing, that it’s easier to write, reason about, and debug but does not replace the need for full integration tests. At the same time it’s important not to lose sight of the fact that the response is the most important thing to check. In short, there is room here for multiple styles and strategies of testing even within the same project.

#### Further Server-Side Test Examples

The framework’s own tests include[many sample tests](https://github.com/spring-projects/spring-framework/tree/master/spring-test/src/test/java/org/springframework/test/web/servlet/samples)intended to demonstrate how to use Spring MVC Test. Browse these examples for further ideas. Also the[spring-mvc-showcase](https://github.com/spring-projects/spring-mvc-showcase)has full test coverage based on Spring MVC Test.

