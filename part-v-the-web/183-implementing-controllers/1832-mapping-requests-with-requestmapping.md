### 18.3.2Mapping Requests With @RequestMapping

You use the`@RequestMapping`annotation to map URLs such as`/appointments`onto an entire class or a particular handler method. Typically the class-level annotation maps a specific request path \(or path pattern\) onto a form controller, with additional method-level annotations narrowing the primary mapping for a specific HTTP request method \("GET", "POST", etc.\) or an HTTP request parameter condition.

The following example from the_Petcare_sample shows a controller in a Spring MVC application that uses this annotation:

```java
@Controller
@RequestMapping("/appointments")
public class AppointmentsController {

	private final AppointmentBook appointmentBook;

	@Autowired
	public AppointmentsController(AppointmentBook appointmentBook) {
		this.appointmentBook = appointmentBook;
	}

	@RequestMapping(method = RequestMethod.GET)
	public Map<String, Appointment> get() {
		return appointmentBook.getAppointmentsForToday();
	}

	@RequestMapping(path = "/{day}", method = RequestMethod.GET)
	public Map<String, Appointment> getForDay(@PathVariable @DateTimeFormat(iso=ISO.DATE) Date day, Model model) {
		return appointmentBook.getAppointmentsForDay(day);
	}

	@RequestMapping(path = "/new", method = RequestMethod.GET)
	public AppointmentForm getNewForm() {
		return new AppointmentForm();
	}

	@RequestMapping(method = RequestMethod.POST)
	public String add(@Valid AppointmentForm appointment, BindingResult result) {
		if (result.hasErrors()) {
			return "appointments/new";
		}
		appointmentBook.addAppointment(appointment);
		return "redirect:/appointments";
	}
}
```

In the above example,`@RequestMapping`is used in a number of places. The first usage is on the type \(class\) level, which indicates that all handler methods in this controller are relative to the`/appointments`path. The`get()`method has a further`@RequestMapping`refinement: it only accepts`GET`requests, meaning that an HTTP`GET`for`/appointments`invokes this method. The`add()`has a similar refinement, and the`getNewForm()`combines the definition of HTTP method and path into one, so that`GET`requests for`appointments/new`are handled by that method.

The`getForDay()`method shows another usage of`@RequestMapping`: URI templates. \(See[the section called “URI Template Patterns”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-ann-requestmapping-uri-templates)\).

A`@RequestMapping`on the class level is not required. Without it, all paths are simply absolute, and not relative. The following example from the_PetClinic_sample application shows a multi-action controller using`@RequestMapping`:

```java
@Controller
public class ClinicController {

	private final Clinic clinic;

	@Autowired
	public ClinicController(Clinic clinic) {
		this.clinic = clinic;
	}

	@RequestMapping("/")
	public void welcomeHandler() {
	}

	@RequestMapping("/vets")
	public ModelMap vetsHandler() {
		return new ModelMap(this.clinic.getVets());
	}

}
```

The above example does not specify`GET`vs.`PUT`,`POST`, and so forth, because`@RequestMapping`maps all HTTP methods by default. Use`@RequestMapping(method=GET)`or`@GetMapping`to narrow the mapping.

#### Composed @RequestMapping Variants

Spring Framework 4.3 introduces the following method-level_composed_variants of the`@RequestMapping`annotation that help to simplify mappings for common HTTP methods and better express the semantics of the annotated handler method. For example, a`@GetMapping`can be read as a`GET@RequestMapping`.

* `@GetMapping`

* `@PostMapping`

* `@PutMapping`

* `@DeleteMapping`

* `@PatchMapping`

The following example shows a modified version of the`AppointmentsController`from the previous section that has been simplified with_composed_`@RequestMapping`annotations.

```java
@Controller
@RequestMapping("/appointments")
public class AppointmentsController {

	private final AppointmentBook appointmentBook;

	@Autowired
	public AppointmentsController(AppointmentBook appointmentBook) {
		this.appointmentBook = appointmentBook;
	}

	@GetMapping
	public Map<String, Appointment> get() {
		return appointmentBook.getAppointmentsForToday();
	}

	@GetMapping("/{day}")
	public Map<String, Appointment> getForDay(@PathVariable @DateTimeFormat(iso=ISO.DATE) Date day, Model model) {
		return appointmentBook.getAppointmentsForDay(day);
	}

	@GetMapping("/new")
	public AppointmentForm getNewForm() {
		return new AppointmentForm();
	}

	@PostMapping
	public String add(@Valid AppointmentForm appointment, BindingResult result) {
		if (result.hasErrors()) {
			return "appointments/new";
		}
		appointmentBook.addAppointment(appointment);
		return "redirect:/appointments";
	}
}
```

#### @Controller and AOP Proxying

In some cases a controller may need to be decorated with an AOP proxy at runtime. One example is if you choose to have`@Transactional`annotations directly on the controller. When this is the case, for controllers specifically, we recommend using class-based proxying. This is typically the default choice with controllers. However if a controller must implement an interface that is not a Spring Context callback \(e.g.`InitializingBean`,`*Aware`, etc\), you may need to explicitly configure class-based proxying. For example with`, change to`.

#### New Support Classes for @RequestMapping methods in Spring MVC 3.1

Spring 3.1 introduced a new set of support classes for`@RequestMapping`methods called`RequestMappingHandlerMapping`and`RequestMappingHandlerAdapter`respectively. They are recommended for use and even required to take advantage of new features in Spring MVC 3.1 and going forward. The new support classes are enabled by default by the MVC namespace and the MVC Java config but must be configured explicitly if using neither. This section describes a few important differences between the old and the new support classes.

Prior to Spring 3.1, type and method-level request mappings were examined in two separate stages — a controller was selected first by the`DefaultAnnotationHandlerMapping`and the actual method to invoke was narrowed down second by the`AnnotationMethodHandlerAdapter`.

With the new support classes in Spring 3.1, the`RequestMappingHandlerMapping`is the only place where a decision is made about which method should process the request. Think of controller methods as a collection of unique endpoints with mappings for each method derived from type and method-level`@RequestMapping`information.

This enables some new possibilities. For once a`HandlerInterceptor`or a`HandlerExceptionResolver`can now expect the Object-based handler to be a`HandlerMethod`, which allows them to examine the exact method, its parameters and associated annotations. The processing for a URL no longer needs to be split across different controllers.

There are also several things no longer possible:

* Select a controller first with a`SimpleUrlHandlerMapping`or`BeanNameUrlHandlerMapping`and then narrow the method based on`@RequestMapping`annotations.

* Rely on method names as a fall-back mechanism to disambiguate between two`@RequestMapping`methods that don’t have an explicit path mapping URL path but otherwise match equally, e.g. by HTTP method. In the new support classes`@RequestMapping`methods have to be mapped uniquely.

* Have a single default method \(without an explicit path mapping\) with which requests are processed if no other controller method matches more concretely. In the new support classes if a matching method is not found a 404 error is raised.

The above features are still supported with the existing support classes. However to take advantage of new Spring MVC 3.1 features you’ll need to use the new support classes.

#### URI Template Patterns

_URI templates_can be used for convenient access to selected parts of a URL in a`@RequestMapping`method.

A URI Template is a URI-like string, containing one or more variable names. When you substitute values for these variables, the template becomes a URI. The[proposed RFC](http://bitworking.org/projects/URI-Templates/)for URI Templates defines how a URI is parameterized. For example, the URI Template`http://www.example.com/users/{userId}`contains the variable_userId_. Assigning the value_fred_to the variable yields`http://www.example.com/users/fred`.

In Spring MVC you can use the`@PathVariable`annotation on a method argument to bind it to the value of a URI template variable:

```java
@GetMapping("/owners/{ownerId}")
public String findOwner(@PathVariable String ownerId, Model model) {
	Owner owner = ownerService.findOwner(ownerId);
	model.addAttribute("owner", owner);
	return "displayOwner";
}
```

The URI Template "/owners/{ownerId}" specifies the variable name`ownerId`. When the controller handles this request, the value of`ownerId`is set to the value found in the appropriate part of the URI. For example, when a request comes in for`/owners/fred`, the value of`ownerId`is`fred`.

| ![](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/tip.png) |
| :--- |
| To process the @PathVariable annotation, Spring MVC needs to find the matching URI template variable by name. You can specify it in the annotation:`*@GetMapping("/owners/{ownerId}")*public String findOwner(**@PathVariable("ownerId")** String theOwner, Model model) { // implementation omitted}`Or if the URI template variable name matches the method argument name you can omit that detail. As long as your code is compiled with debugging information or the`-parameters`compiler flag on Java 8, Spring MVC will match the method argument name to the URI template variable name:`*@GetMapping("/owners/{ownerId}")*public String findOwner(**@PathVariable** String ownerId, Model model) { // implementation omitted}` |

A method can have any number of`@PathVariable`annotations:

```java
@GetMapping("/owners/{ownerId}/pets/{petId}")
public String findPet(@PathVariable String ownerId, @PathVariable String petId, Model model) {
	Owner owner = ownerService.findOwner(ownerId);
	Pet pet = owner.getPet(petId);
	model.addAttribute("pet", pet);
	return "displayPet";
}
```

When a`@PathVariable`annotation is used on a`Map`argument, the map is populated with all URI template variables.

A URI template can be assembled from type and method level_@RequestMapping_annotations. As a result the`findPet()`method can be invoked with a URL such as`/owners/42/pets/21`.

```java
@Controller
@RequestMapping("/owners/{ownerId}")
public class RelativePathUriTemplateController {

	@RequestMapping("/pets/{petId}")
	public void findPet(@PathVariable String ownerId, @PathVariable String petId, Model model) {
		// implementation omitted
	}

}
```

A`@PathVariable`argument can be of_any simple type_such as`int`,`long`,`Date`, etc. Spring automatically converts to the appropriate type or throws a`TypeMismatchException`if it fails to do so. You can also register support for parsing additional data types. See[the section called “Method Parameters And Type Conversion”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-ann-typeconversion)and[the section called “Customizing WebDataBinder initialization”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-ann-webdatabinder).

#### URI Template Patterns with Regular Expressions

Sometimes you need more precision in defining URI template variables. Consider the URL`"/spring-web/spring-web-3.0.5.jar"`. How do you break it down into multiple parts?

The`@RequestMapping`annotation supports the use of regular expressions in URI template variables. The syntax is`{varName:regex}`where the first part defines the variable name and the second - the regular expression. For example:

```java
@RequestMapping("/spring-web/{symbolicName:[a-z-]+}-{version:\\d\\.\\d\\.\\d}{extension:\\.[a-z]+}")
public void handle(@PathVariable String version, @PathVariable String extension) {
	// ...
}
```

####  Path Patterns

In addition to URI templates, the`@RequestMapping`annotation and all_composed_`@RequestMapping`variants also support Ant-style path patterns \(for example,`/myPath/*.do`\). A combination of URI template variables and Ant-style globs is also supported \(e.g.`/owners/*/pets/{petId}`\).

#### Path Pattern Comparison

When a URL matches multiple patterns, a sort is used to find the most specific match.

A pattern with a lower count of URI variables and wild cards is considered more specific. For example`/hotels/{hotel}/*`has 1 URI variable and 1 wild card and is considered more specific than`/hotels/{hotel}/**`which as 1 URI variable and 2 wild cards.

If two patterns have the same count, the one that is longer is considered more specific. For example`/foo/bar*`is longer and considered more specific than`/foo/*`.

When two patterns have the same count and length, the pattern with fewer wild cards is considered more specific. For example`/hotels/{hotel}`is more specific than`/hotels/*`.

There are also some additional special rules:

* The**default mapping pattern**`/**`is less specific than any other pattern. For example`/api/{a}/{b}/{c}`is more specific.

* A**prefix pattern**such as`/public/**`is less specific than any other pattern that doesn’t contain double wildcards. For example`/public/path3/{a}/{b}/{c}`is more specific.

For the full details see`AntPatternComparator`in`AntPathMatcher`. Note that the PathMatcher can be customized \(see[Section18.16.11, “Path Matching”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-config-path-matching)in the section on configuring Spring MVC\).

#### Path Patterns with Placeholders

Patterns in`@RequestMapping`annotations support`${…​}`placeholders against local properties and/or system properties and environment variables. This may be useful in cases where the path a controller is mapped to may need to be customized through configuration. For more information on placeholders, see the javadocs of the`PropertyPlaceholderConfigurer`class.

#### Suffix Pattern Matching

By default Spring MVC performs`".*"`suffix pattern matching so that a controller mapped to`/person`is also implicitly mapped to`/person.*`. This makes it easy to request different representations of a resource through the URL path \(e.g.`/person.pdf`,`/person.xml`\).

Suffix pattern matching can be turned off or restricted to a set of path extensions explicitly registered for content negotiation purposes. This is generally recommended to minimize ambiguity with common request mappings such as`/person/{id}`where a dot might not represent a file extension, e.g.`/person/joe@email.com`vs`/person/joe@email.com.json`. Furthermore as explained in the note below suffix pattern matching as well as content negotiation may be used in some circumstances to attempt malicious attacks and there are good reasons to restrict them meaningfully.

See[Section18.16.11, “Path Matching”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-config-path-matching)for suffix pattern matching configuration and also[Section18.16.6, “Content Negotiation”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-config-content-negotiation)for content negotiation configuration.

#### Suffix Pattern Matching and RFD

Reflected file download \(RFD\) attack was first described in a[paper by Trustwave](https://www.trustwave.com/Resources/SpiderLabs-Blog/Reflected-File-Download---A-New-Web-Attack-Vector/)in 2014. The attack is similar to XSS in that it relies on input \(e.g. query parameter, URI variable\) being reflected in the response. However instead of inserting JavaScript into HTML, an RFD attack relies on the browser switching to perform a download and treating the response as an executable script if double-clicked based on the file extension \(e.g. .bat, .cmd\).

In Spring MVC`@ResponseBody`and`ResponseEntity`methods are at risk because they can render different content types which clients can request including via URL path extensions. Note however that neither disabling suffix pattern matching nor disabling the use of path extensions for content negotiation purposes alone are effective at preventing RFD attacks.

For comprehensive protection against RFD, prior to rendering the response body Spring MVC adds a`Content-Disposition:inline;filename=f.txt`header to suggest a fixed and safe download file. This is done only if the URL path contains a file extension that is neither whitelisted nor explicitly registered for content negotiation purposes. However it may potentially have side effects when URLs are typed directly into a browser.

Many common path extensions are whitelisted by default. Furthermore REST API calls are typically not meant to be used as URLs directly in browsers. Nevertheless applications that use custom`HttpMessageConverter`implementations can explicitly register file extensions for content negotiation and the Content-Disposition header will not be added for such extensions. See[Section18.16.6, “Content Negotiation”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-config-content-negotiation).

| ![](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png) |
| :--- |
| This was originally introduced as part of work for[CVE-2015-5211](https://pivotal.io/security/cve-2015-5211). Below are additional recommendations from the report:Encode rather than escape JSON responses. This is also an OWASP XSS recommendation. For an example of how to do that with Spring see[spring-jackson-owasp](https://github.com/rwinch/spring-jackson-owasp).Configure suffix pattern matching to be turned off or restricted to explicitly registered suffixes only.Configure content negotiation with the properties "useJaf" and "ignoreUnknownPathExtensions" set to false which would result in a 406 response for URLs with unknown extensions. Note however that this may not be an option if URLs are naturally expected to have a dot towards the end.Add`X-Content-Type-Options: nosniff`header to responses. Spring Security 4 does this by default. |

#### Matrix Variables

The URI specification[RFC 3986](https://tools.ietf.org/html/rfc3986#section-3.3)defines the possibility of including name-value pairs within path segments. There is no specific term used in the spec. The general "URI path parameters" could be applied although the more unique["Matrix URIs"](https://www.w3.org/DesignIssues/MatrixURIs.html), originating from an old post by Tim Berners-Lee, is also frequently used and fairly well known. Within Spring MVC these are referred to as matrix variables.

Matrix variables can appear in any path segment, each matrix variable separated with a ";" \(semicolon\). For example:`"/cars;color=red;year=2012"`. Multiple values may be either "," \(comma\) separated`"color=red,green,blue"`or the variable name may be repeated`"color=red;color=green;color=blue"`.

If a URL is expected to contain matrix variables, the request mapping pattern must represent them with a URI template. This ensures the request can be matched correctly regardless of whether matrix variables are present or not and in what order they are provided.

Below is an example of extracting the matrix variable "q":

```java
// GET /pets/42;q=11;r=22

@GetMapping("/pets/{petId}")
public void findPet(@PathVariable String petId, @MatrixVariable int q) {

	// petId == 42
	// q == 11

}
```

Since all path segments may contain matrix variables, in some cases you need to be more specific to identify where the variable is expected to be:

```java
// GET /owners/42;q=11/pets/21;q=22

@GetMapping("/owners/{ownerId}/pets/{petId}")
public void findPet(
		@MatrixVariable(name="q", pathVar="ownerId") int q1,
		@MatrixVariable(name="q", pathVar="petId") int q2) {

	// q1 == 11
	// q2 == 22

}
```

A matrix variable may be defined as optional and a default value specified:

```java
// GET /pets/42

@GetMapping("/pets/{petId}")
public void findPet(@MatrixVariable(required=false, defaultValue="1") int q) {

	// q == 1

}
```

All matrix variables may be obtained in a Map:

```java
// GET /owners/42;q=11;r=12/pets/21;q=22;s=23

@GetMapping("/owners/{ownerId}/pets/{petId}")
public void findPet(
		@MatrixVariable MultiValueMap<String, String> matrixVars,
		@MatrixVariable(pathVar="petId"") MultiValueMap<String, String> petMatrixVars) {

	// matrixVars: ["q" : [11,22], "r" : 12, "s" : 23]
	// petMatrixVars: ["q" : 11, "s" : 23]

}
```

Note that to enable the use of matrix variables, you must set the`removeSemicolonContent`property of`RequestMappingHandlerMapping`to`false`. By default it is set to`true`.

| ![](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/tip.png) |
| :--- |
| The MVC Java config and the MVC namespace both provide options for enabling the use of matrix variables.If you are using Java config, The[Advanced Customizations with MVC Java Config](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-config-advanced-java)section describes how the`RequestMappingHandlerMapping`can be customized.In the MVC namespace, the```element has an``enable-matrix-variables`attribute that should be set to`true`. By default it is set to`false`.`\` |

####  Consumable Media Types

You can narrow the primary mapping by specifying a list of consumable media types. The request will be matched only if the`Content-Type`request header matches the specified media type. For example:

```java
@PostMapping(path = "/pets", consumes = "application/json")
public void addPet(@RequestBody Pet pet, Model model) {
	// implementation omitted
}
```

Consumable media type expressions can also be negated as in`!text/plain`to match to all requests other than those with`Content-Type`of`text/plain`. Also consider using constants provided in`MediaType`such as`APPLICATION_JSON_VALUE`and`APPLICATION_JSON_UTF8_VALUE`.

| ![](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/tip.png) |
| :--- |
| The_consumes_condition is supported on the type and on the method level. Unlike most other conditions, when used at the type level, method-level consumable types override rather than extend type-level consumable types. |

#### Producible Media Types

You can narrow the primary mapping by specifying a list of producible media types. The request will be matched only if the`Accept`request header matches one of these values. Furthermore, use of the_produces_condition ensures the actual content type used to generate the response respects the media types specified in the_produces_condition. For example:

```java
@GetMapping(path = "/pets/{petId}", produces = MediaType.APPLICATION_JSON_UTF8_VALUE)
@ResponseBody
public Pet getPet(@PathVariable String petId, Model model) {
	// implementation omitted
}
```

| ![](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png) |
| :--- |
| Be aware that the media type specified in the_produces_condition can also optionally specify a character set. For example, in the code snippet above we specify the same media type than the default one configured in`MappingJackson2HttpMessageConverter`, including the`UTF-8`charset. |

Just like with_consumes_, producible media type expressions can be negated as in`!text/plain`to match to all requests other than those with an`Accept`header value of`text/plain`. Also consider using constants provided in`MediaType`such as`APPLICATION_JSON_VALUE`and`APPLICATION_JSON_UTF8_VALUE`.

| ![](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/tip.png) |
| :--- |
| The_produces_condition is supported on the type and on the method level. Unlike most other conditions, when used at the type level, method-level producible types override rather than extend type-level producible types. |

#### Request Parameters and Header Values

You can narrow request matching through request parameter conditions such as`"myParam"`,`"!myParam"`, or`"myParam=myValue"`. The first two test for request parameter presence/absence and the third for a specific parameter value. Here is an example with a request parameter value condition:

```java
@Controller
@RequestMapping("/owners/{ownerId}")
public class RelativePathUriTemplateController {

	@GetMapping(path = "/pets/{petId}", params = "myParam=myValue")
	public void findPet(@PathVariable String ownerId, @PathVariable String petId, Model model) {
		// implementation omitted
	}

}
```

The same can be done to test for request header presence/absence or to match based on a specific request header value:

```java
@Controller
@RequestMapping("/owners/{ownerId}")
public class RelativePathUriTemplateController {

	@GetMapping(path = "/pets", headers = "myHeader=myValue")
	public void findPet(@PathVariable String ownerId, @PathVariable String petId, Model model) {
		// implementation omitted
	}

}
```

| ![](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/tip.png) |
| :--- |
| Although you can match to_Content-Type_and_Accept_header values using media type wild cards \(for example_"content-type=text/\*"_will match to_"text/plain"_and_"text/html"_\), it is recommended to use the_consumes_and_produces_conditions respectively instead. They are intended specifically for that purpose. |

#### HTTP HEAD and HTTP OPTIONS

`@RequestMapping`methods mapped to "GET" are also implicitly mapped to "HEAD", i.e. there is no need to have "HEAD" explicitly declared. An HTTP HEAD request is processed as if it were an HTTP GET except instead of writing the body only the number of bytes are counted and the "Content-Length" header set.

`@RequestMapping`methods have built-in support for HTTP OPTIONS. By default an HTTP OPTIONS request is handled by setting the "Allow" response header to the HTTP methods explicitly declared on all`@RequestMapping`methods with matching URL patterns. When no HTTP methods are explicitly declared the "Allow" header is set to "GET,HEAD,POST,PUT,PATCH,DELETE,OPTIONS". Ideally always declare the HTTP method\(s\) that an`@RequestMapping`method is intended to handle, or alternatively use one of the dedicated_composed_`@RequestMapping`variants \(see[the section called “Composed @RequestMapping Variants”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-ann-requestmapping-composed)\).

Although not necessary an`@RequestMapping`method can be mapped to and handle either HTTP HEAD or HTTP OPTIONS, or both.

  


