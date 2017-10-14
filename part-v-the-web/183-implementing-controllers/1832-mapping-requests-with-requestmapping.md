### 18.3.2使用@RequestMapping映射请求

您可以使用`@RequestMapping`注释将诸如`/appointments`的URL映射到整个类或特定的处理程序方法。 通常，类级注释将特定的请求路径（或路径模式）映射到表单控制器上，其他方法级注释缩小了特定HTTP请求方法（“GET”，“POST”等）的主映射，或 HTTP请求参数条件。

Petcare示例中的以下示例显示了使用此注释的Spring MVC应用程序中的控制器：

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

在上面的例子中，`@RequestMapping`用在很多地方。 第一个用法是类型（类）级别，这表示此控制器中的所有处理程序方法都相对于`/appointments`路径。 `get()`方法还有一个@RequestMapping细化：它只接受GET请求，这意味着`/appointments` 的HTTP `GET`调用此方法。`add()`有一个类似的细化，`getNewForm()`将HTTP方法和路径的定义组合成一个，以便通过该方法处理`appointments/new`的GET请求。

`getForDay()`方法显示了`@RequestMapping`：URI模板的另一种用法。 （参见[“URI模板模式”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-ann-requestmapping-uri-templates)一节）。

类级别上的`@RequestMapping`不是必需的。 没有它，所有的路径都是绝对的，而不是相对的。 PetClinic示例应用程序的以下示例显示了使用`@RequestMapping`的多操作控制器：

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

上述示例不指定`GET`与`PUT`，`POST`等，因为`@RequestMapping`默认映射所有HTTP方法。 使用`@RequestMapping(method=GET)`或`@GetMapping`来缩小映射。

#### 组合@RequestMapping变体

Spring Framework 4.3引入了`@RequestMapping`注释的以下方法级组合变体，有助于简化常见HTTP方法的映射，并更好地表达注释处理程序方法的语义。 例如，`@GetMapping`可以被读取为GET `@RequestMapping`。

* `@GetMapping`

* `@PostMapping`

* `@PutMapping`

* `@DeleteMapping`

* `@PatchMapping`

以下示例显示了使用已组合的`@RequestMapping`注释简化的上一节中的`AppointmentsController`的修改版本。

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

#### @Controller 和AOP 代理

在某些情况下，控制器可能需要在运行时用AOP代理进行装饰。 一个例子是如果您选择在控制器上直接使用`@Transactional`注释。 在这种情况下，对于控制器，我们建议使用基于类的代理。 这通常是控制器的默认选项。 但是，如果控制器必须实现不是Spring Context回调的接口（例如`InitializingBean`，`* Aware`等），则可能需要显式配置基于类的代理。 例如，使用

`<tx：annotation-driven />`，更改为`<tx：annotation-driven proxy-target-class =“true”/>`。

#### Spring MVC 3.1中的@RequestMapping方法的新支持类

Spring 3.1分别为`@RequestMapping`方法引入了一组新的支持类，分别叫做`RequestMappingHandlerMapping`和`RequestMappingHandlerAdapter`。它们被推荐使用，甚至需要利用Spring MVC 3.1中的新功能和未来。默认情况下，MVC命名空间和MVC Java配置启用新的支持类，但是如果不使用，则必须显式配置。本节介绍旧支持类和新支持类之间的一些重要区别。

在Spring 3.1之前，类型和方法级请求映射在两个单独的阶段进行了检查 – 首先由`DefaultAnnotationHandlerMapping`选择一个控制器，并且实际的调用方法被`AnnotationMethodHandlerAdapter`缩小。

使用Spring 3.1中的新支持类，`RequestMappingHandlerMapping`是唯一可以决定哪个方法应该处理请求的地方。将控制器方法作为从类型和方法级`@RequestMapping`信息派生的每个方法的映射的唯一端点的集合。

这使得一些新的可能性。一旦`HandlerInterceptor`或`HandlerExceptionResolver`现在可以期望基于对象的处理程序是`HandlerMethod`，它允许它们检查确切的方法，其参数和关联的注释。 URL的处理不再需要跨不同的控制器进行拆分。

还有下面几件事情已经不复存在了：

* 首先使用`SimpleUrlHandlerMapping`或`BeanNameUrlHandlerMapping`选择控制器，然后基于`@RequestMapping`注释来缩小方法。

* 依赖于方法名称作为一种落后机制，以消除两个`@RequestMapping`方法之间的差异，这两个方法没有明确的路径映射URL路径， 通过HTTP方法。 在新的支持类中，`@RequestMapping`方法必须被唯一地映射。

* 如果没有其他控制器方法更具体地匹配，请使用单个默认方法（无显式路径映射）处理请求。 在新的支持类中，如果找不到匹配方法，则会引发404错误。

上述功能仍然支持现有的支持类。 不过要利用新的Spring MVC 3.1功能，您需要使用新的支持类。

#### URI 模版模式

可以使用URI模板方便地访问`@RequestMapping`方法中URL的所选部分。

URI模板是一个类似URI的字符串，包含一个或多个变量名称。 当您替换这些变量的值时，模板将成为一个URI。 所提出的RFC模板RFC定义了URI如何参数化。 例如，URI模板`http://www.example.com/users/{userId}`包含变量userId。 将`fred`的值分配给变量会得到`http://www.example.com/users/fred`。

在Spring MVC中，您可以使用方法参数上的`@PathVariable`注释将其绑定到URI模板变量的值：

```java
@GetMapping("/owners/{ownerId}")
public String findOwner(@PathVariable String ownerId, Model model) {
    Owner owner = ownerService.findOwner(ownerId);
    model.addAttribute("owner", owner);
    return "displayOwner";
}
```

URI模板“`/ owners / {ownerId}`”指定变量名`ownerId`。 当控制器处理此请求时，`ownerId`的值将设置为在URI的适当部分中找到的值。 例如，当`/owner/fred`出现请求时，`ownerId`的值为`fred`。

| ![](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/tip.png) |
| :--- |
| 要处理`@PathVariable`注释，Spring MVC需要按名称找到匹配的URI模板变量。 您可以在注释中指定它：`*@GetMapping("/owners/{ownerId}")*public String findOwner(**@PathVariable("ownerId")** String theOwner, Model model) { // implementation omitted}`或者如果URI模板变量名称与方法参数名称匹配，则可以省略该详细信息。 只要您的代码使用Java 8编译调试信息或参数编译器标志，Spring MVC将将方法参数名称与URI模板变量名称相匹配：`*@GetMapping("/owners/{ownerId}")*public String findOwner(**@PathVariable** String ownerId, Model model) { // implementation omitted}` |

一个方法可以有任何数量的`@PathVariable`注释：

```java
@GetMapping("/owners/{ownerId}/pets/{petId}")
public String findPet(@PathVariable String ownerId, @PathVariable String petId, Model model) {
    Owner owner = ownerService.findOwner(ownerId);
    Pet pet = owner.getPet(petId);
    model.addAttribute("pet", pet);
    return "displayPet";
}
```

当在`Map <String，String>`参数上使用`@PathVariable`注释时，映射将填充所有URI模板变量。

URI模板可以从类型和方法级别`@RequestMapping`注释中进行组合。 因此，可以使用`/owners/42/pets/21`等URL调用`findPet()`方法。

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

`@PathVariable`参数可以是\_\_any简单的type\_such，如`int`，`long`，`Date`等。如果没有这样做，Spring将自动转换为适当的类型或抛出TypeMismatchException。 您还可以注册解析附加数据类型的支持。 Seethe部分称为[“方法参数和类型转换”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-ann-typeconversion)和[“定制WebDataBinder初始化”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-ann-webdatabinder)一节。

#### 具有正则表达式的URI模板模式

有时您需要更精确地定义URI模板变量。 考虑URL`“/spring-web/spring-web-3.0.5.jar”`。 你怎么把它分解成多个部分？

`@RequestMapping`注释支持在URI模板变量中使用正则表达式。 语法是`{varName：regex}`，其中第一部分定义了变量名，第二部分定义了正则表达式。 例如：

```java
@RequestMapping("/spring-web/{symbolicName:[a-z-]+}-{version:\\d\\.\\d\\.\\d}{extension:\\.[a-z]+}")
public void handle(@PathVariable String version, @PathVariable String extension) {
    // ...
}
```

#### 路径模式

除了URI模板之外，`@RequestMapping`注释和所有组合的`@RequestMapping`变体也支持Ant样式的路径模式（例如`/myPath/*.do`）。 还支持URI模板变量和Ant-style glob的组合（例如`/owners/*/pets/{petId}`）。

#### 路径模式比较

当URL匹配多个模式时，使用排序来查找最具体的匹配。

具有较低数量URI变量和通配符的模式被认为更具体。 例如/`hotels/{hotel}/*`具有1个URI变量和1个通配符，被认为比`/hotels/{hotel}/**`更具体，其中1个URI变量和2个通配符。

如果两个模式具有相同的计数，那么较长的模式被认为更具体。 例如`/foo/bar*`比较长，被认为比`/foo/*`更具体。

当两个模式具有相同的计数和长度时，具有较少通配符的模式被认为更具体。 例如`/hotels/ {hotel}`比`/hotels/*`更具体。

下面有些额外增加的特殊的规则：

* **默认映射模式**`/**`比任何其他模式都要小。 例如`/api/{a}/{b}/{c}`更具体。

* 诸如`/public/**`之类的**前缀模式**比不包含双通配符的任何其他模式都不那么具体。 例如`/public/path3/{a}/{b}/{c}`更具体。

有关详细信息，请参阅AntPathMatcher中的AntPatternComparator。 请注意，可以自定义PathMatcher（参见[Section 18.16.11, “Path Matching”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-config-path-matching)\).

#### 具有占位符的路径模式

`@RequestMapping`注释中的模式支持对本地属性and/or系统属性和环境变量的`${…}`占位符。 在将控制器映射到的路径可能需要通过配置进行定制的情况下，这可能是有用的。 有关占位符的更多信息，请参阅`PropertyPlaceholderConfigurer`类的javadocs。

#### 后缀模式匹配

默认情况下，Spring MVC执行`".*"`后缀模式匹配，以便映射到`/person`的控制器也隐式映射到`/person.*`。这使得通过URL路径（例如`/person.pdf`,`/person.xml`）可以轻松地请求资源的不同表示。

后缀模式匹配可以关闭或限制为一组明确注册用于内容协商的路径扩展。通常建议通过诸如`/person/{id}`之类的常见请求映射来减少歧义，其中点可能不表示文件扩展名，例如`/person/joe@email.com` vs `/person/joe@email.com.json`。此外，如下面的说明中所解释的，后缀模式匹配以及内容协商可能在某些情况下用于尝试恶意攻击，并且有充分的理由有意义地限制它们。

有关后缀模式匹配配置，请参见[Section18.16.11, “Path Matching”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-config-path-matching)，内容协商配置[Section18.16.6, “Content Negotiation”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-config-content-negotiation)。

#### 后缀模式匹配和RFD

反射文件下载（RFD）攻击是由Trustwave在2014年的一篇论文中首次描述的。攻击类似于XSS，因为它依赖于响应中反映的输入（例如查询参数，URI变量）。然而，不是将JavaScript插入到HTML中，如果基于文件扩展名（例如.bat，.cmd）双击，则RFD攻击依赖于浏览器切换来执行下载并将响应视为可执行脚本。

在Spring MVC `@ResponseBody`和`ResponseEntity`方法存在风险，因为它们可以呈现客户端可以通过URL路径扩展请求的不同内容类型。但是请注意，单独禁用后缀模式匹配或禁用仅用于内容协商的路径扩展都可以有效地防止RFD攻击。

为了全面保护RFD，在呈现响应体之前，Spring MVC添加了`Content-Disposition:inline;filename=f.txt`头来建议一个固定和安全的下载文件。只有当URL路径包含既不是白名单的文件扩展名，也没有明确注册用于内容协商的目的，这是完成的。但是，当URL直接输入浏览器时，可能会产生副作用。

许多常见的路径扩展名默认为白名单。此外，REST API调用通常不是直接用于浏览器中的URL。然而，使用自定义`HttpMessageConverter`实现的应用程序可以明确地注册用于内容协商的文件扩展名，并且不会为此类扩展添加Content-Disposition头。见第18.16.6节[“Content Negotiation”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-config-content-negotiation)。

| ![](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png) |
| :--- |
| 这是[CVE-2015-5211](https://pivotal.io/security/cve-2015-5211)工作的一部分。 以下是报告中的其他建议：1、编码而不是转义JSON响应。 这也是OWASP XSS的建议。 有关Spring的例子，请参阅[spring-jackson-owasp](https://github.com/rwinch/spring-jackson-owasp) 2、将后缀模式匹配配置为关闭或仅限于明确注册的后缀。3、配置使用属性“useJaf”和“ignoreUnknownPathExtensions”设置为false的内容协商，这将导致具有未知扩展名的URL的406响应。 但是请注意，如果URL自然希望有一个结束点，这可能不是一个选择4、添加`X-Content-Type-Options：nosniff`头到响应。 Spring Security 4默认情况下执行此操作。 |

#### 矩阵变量

URI规范[RFC 3986](https://tools.ietf.org/html/rfc3986#section-3.3)定义了在路径段中包含名称 – 值对的可能性。规格中没有使用具体术语。可以应用一般的“URI路径参数”，尽管来自Tim Berners-Lee的旧帖子的更独特的["Matrix URIs"](https://www.w3.org/DesignIssues/MatrixURIs.html)也经常被使用并且是相当熟知的。在Spring MVC中，这些被称为矩阵变量。

矩阵变量可以出现在任何路径段中，每个矩阵变量用“;”分隔（分号）。例如：`"/cars;color=red;year=2012"`。多个值可以是“，”（逗号）分隔`"color=red,green,blue"`，或者变量名称可以重复`"color=red;color=green;color=blue"`。

如果URL预期包含矩阵变量，则请求映射模式必须使用URI模板来表示它们。这确保了请求可以正确匹配，无论矩阵变量是否存在，以及它们提供什么顺序。

以下是提取矩阵变量“q”的示例：

```java
// GET /pets/42;q=11;r=22

@GetMapping("/pets/{petId}")
public void findPet(@PathVariable String petId, @MatrixVariable int q) {

    // petId == 42
    // q == 11

}
```

由于所有路径段都可能包含矩阵变量，因此在某些情况下，您需要更具体地确定变量预期位于何处：

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

矩阵变量可以定义为可选参数，并指定一个默认值：

```java
// GET /pets/42

@GetMapping("/pets/{petId}")
public void findPet(@MatrixVariable(required=false, defaultValue="1") int q) {

    // q == 1

}
```

所有矩阵变量可以在Map中获得：

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

请注意，为了使用矩阵变量，您必须将`RequestMappingHandlerMapping`的`removeSemicolonContent`属性设置为`false`。 默认设置为`true`。

> MVC Java配置和MVC命名空间都提供了使用矩阵变量的选项。
>
> 如果您使用Java配置，使用MVC Java Config的高级自定义部分将介绍如何自定义RequestMappingHandlerMapping。
>
> 在MVC命名空间中，`<mvc：annotation-driven>`元素具有一个应该设置为`true`的`enable-matrix-variables`属性。 默认情况下设置为`false`。
>
> ```
>     <?xml version="1.0" encoding="UTF-8"?>
>     <beans xmlns="http://www.springframework.org/schema/beans"
>         xmlns:mvc="http://www.springframework.org/schema/mvc"
>         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
>         xsi:schemaLocation="
>             http://www.springframework.org/schema/beans
>             http://www.springframework.org/schema/beans/spring-beans.xsd
>             http://www.springframework.org/schema/mvc
>             http://www.springframework.org/schema/mvc/spring-mvc.xsd">
>
>         <mvc:annotation-driven enable-matrix-variables="true"/>
>
>     </beans>
> ```

#### Consumable Media 类型

您可以通过指定consumable media类型的列表来缩小主要映射。 只有当`Content-Type`请求头与指定的媒体类型匹配时，才会匹配该请求。 例如：

```java
@PostMapping(path = "/pets", consumes = "application/json")
public void addPet(@RequestBody Pet pet, Model model) {
    // implementation omitted
}
```

consumable media类型表达式也可以在`!text/plain`中否定，以匹配除`Content-Type` 或 `text/plain`之外的所有请求。 还要考虑使用`MediaType`中提供的常量，例如`APPLICATION_JSON_VALUE`和A`PPLICATION_JSON_UTF8_VALUE`。

| ![](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/tip.png) |
| :--- |
| 在类型和方法级别上支持\_\_consumes\_condition。 与大多数其他条件不同，当在类型级别使用时，方法级consumable types将覆盖而不是扩展类型级别的consumable types。 |

#### Producible Media 类型

您可以通过指定producible media类型列表来缩小主要映射。 只有当`Accept`请求头匹配这些值之一时，才会匹配该请求。 此外，使用产生条件确保用于产生响应的实际内容类型与产生条件中指定的媒体类型相关。 例如：

```java
@GetMapping(path = "/pets/{petId}", produces = MediaType.APPLICATION_JSON_UTF8_VALUE)
@ResponseBody
public Pet getPet(@PathVariable String petId, Model model) {
    // implementation omitted
}
```

| ![](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png) |
| :--- |
| 在\_products\_condition中指定的媒体类型也可以选择指定一个字符集。 例如，在上面的代码片段中，我们指定与默认配置在`MappingJackson2HttpMessageConverter`中的介质类型相同的介质类型，包括`UTF-8` charset。 |

就像消费一样，producible media类型表达式可以被否定为`!text/plain`，以匹配除了`Accept`头文件值为`text/plain`的所有请求。 还要考虑使用`MediaType`中提供的常量，例如`APPLICATION_JSON_VALUE`和`APPLICATION_JSON_UTF8_VALUE`。

| ![](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/tip.png) |
| :--- |
| 在类型和方法级别上支持\_\_produces\_condition。 与大多数其他条件不同，当在类型级别使用时，方法级producible类型将覆盖而不是扩展类型级producible类型。 |

#### 请求参数和头部值

您可以通过请求参数条件（如`"myParam"`，`"!myParam"`或`"myParam=myValue"`）缩小请求匹配。 前两个测试请求参数存在/不存在，第三个为特定参数值。 下面是一个请求参数值条件的例子：

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

也可以根据特定的请求头值来测试请求头存在/不存在或匹配：

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
| 虽然可以使用媒体类型通配符匹配toContent-Type\_and\_Accept\_header值（例如“content-type = text / \*”将匹配“text / plain”和“text / html”\_），但建议分别使用\_onsumes\_and\_produces\_cignitions。 它们专门用于此目的。 |

#### HTTP 头部和 HTTP 可选项

映射到“GET”的`@RequestMapping`方法也隐式映射到“HEAD”，即不需要明确声明“HEAD”。处理HTTP HEAD请求就像是HTTP GET一样，而不是仅写入正文，仅计数字节数，并设置“Content-Length”头。

`@RequestMapping`方法内置支持HTTP选项。默认情况下，通过将所有`@RequestMapping`方法上显式声明的具有匹配URL模式的HTTP方法设置为“允许”响应头来处理HTTP OPTIONS请求。当没有明确声明HTTP方法时，“允许”标题设置为“GET，HEAD，POST，PUT，PATCH，DELETE，OPTIONS”。理想地总是声明`@RequestMapping`方法要处理的HTTP方法，或者使用专用的组合`@RequestMapping`变体之一（参见[“Composed @RequestMapping Variants”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-ann-requestmapping-composed)一节）。

虽然不需要`@RequestMapping`方法可以映射到HTTP HEAD或HTTP选项，也可以两者兼容。

