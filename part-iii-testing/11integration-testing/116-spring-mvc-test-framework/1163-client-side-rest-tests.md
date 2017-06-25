### 11.6.3 Client-Side REST Tests

Client-side tests can be used to test code that internally uses the`RestTemplate`. The idea is to declare expected requests and to provide "stub" responses so that you can focus on testing the code in isolation, i.e. without running a server. Here is an example:

```java
RestTemplate restTemplate = new RestTemplate();

MockRestServiceServer mockServer = MockRestServiceServer.bindTo(restTemplate).build();
mockServer.expect(requestTo("/greeting")).andRespond(withSuccess());

// Test code that uses the above RestTemplate ...

mockServer.verify();
```

In the above example,`MockRestServiceServer`, the central class for client-side REST tests, configures the`RestTemplate`with a custom`ClientHttpRequestFactory`that asserts actual requests against expectations and returns "stub" responses. In this case we expect a request to "/greeting" and want to return a 200 response with "text/plain" content. We could define as additional expected requests and stub responses as needed. When expected requests and stub responses are defined, the`RestTemplate`can be used in client-side code as usual. At the end of testing`mockServer.verify()`can be used to verify that all expectations have been satisfied.

By default requests are expected in the order in which expectations were declared. You can set the`ignoreExpectOrder`option when building the server in which case all expectations are checked \(in order\) to find a match for a given request. That means requests are allowed to come in any order. Here is an example:

```java
server = MockRestServiceServer.bindTo(restTemplate).ignoreExpectOrder(true).build();
```

Even with unordered requests by default each request is allowed to execute once only. The`expect`method provides an overloaded variant that accepts an`ExpectedCount`argument that specifies a count range, e.g.`once`,`manyTimes`,`max`,`min`,`between`, and so on. Here is an example:

```java
RestTemplate restTemplate = new RestTemplate();

MockRestServiceServer mockServer = MockRestServiceServer.bindTo(restTemplate).build();
mockServer.expect(times(2), requestTo("/foo")).andRespond(withSuccess());
mockServer.expect(times(3), requestTo("/bar")).andRespond(withSuccess());

// ...

mockServer.verify();
```

Note that when`ignoreExpectOrder`is not set \(the default\), and therefore requests are expected in order of declaration, then that order only applies to the first of any expected request. For example if "/foo" is expected 2 times followed by "/bar" 3 times, then there should be a request to "/foo" before there is a request to "/bar" but aside from that subsequent "/foo" and "/bar" requests can come at any time.

As an alternative to all of the above the client-side test support also provides a`ClientHttpRequestFactory`implementation that can be configured into a`RestTemplate`to bind it to a`MockMvc`instance. That allows processing requests using actual server-side logic but without running a server. Here is an example:

```java
MockMvc mockMvc = MockMvcBuilders.webAppContextSetup(this.wac).build();
this.restTemplate = new RestTemplate(new MockMvcClientHttpRequestFactory(mockMvc));

// Test code that uses the above RestTemplate ...

mockServer.verify();
```

#### Static Imports

Just like with server-side tests, the fluent API for client-side tests requires a few static imports. Those are easy to find by searching_"MockRest\*"_. Eclipse users should add`"MockRestRequestMatchers.*"`and`"MockRestResponseCreators.*"`as "favorite static members" in the Eclipse preferences under_Java → Editor → Content Assist → Favorites_. That allows using content assist after typing the first character of the static method name. Other IDEs \(e.g. IntelliJ\) may not require any additional configuration. Just check the support for code completion on static members.

#### Further Examples of Client-side REST Tests

Spring MVC Test’s own tests include[example tests](https://github.com/spring-projects/spring-framework/tree/master/spring-test/src/test/java/org/springframework/test/web/client/samples)of client-side REST tests.

## 



