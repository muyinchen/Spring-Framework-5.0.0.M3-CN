## 18.3Implementing Controllers

Controllers provide access to the application behavior that you typically define through a service interface. Controllers interpret user input and transform it into a model that is represented to the user by the view. Spring implements a controller in a very abstract way, which enables you to create a wide variety of controllers.

Spring 2.5 introduced an annotation-based programming model for MVC controllers that uses annotations such as`@RequestMapping`,`@RequestParam`,`@ModelAttribute`, and so on. Controllers implemented in this style do not have to extend specific base classes or implement specific interfaces. Furthermore, they do not usually have direct dependencies on Servlet APIs, although you can easily configure access to Servlet facilities if needed.

| ![](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/tip.png) |
| :--- |
| Available in the[spring-projects Org on Github](https://github.com/spring-projects/), a number of web applications leverage the annotation support described in this section including_MvcShowcase_,_MvcAjax_,_MvcBasic_,_PetClinic_,_PetCare_, and others. |

```java
@Controller
public class HelloWorldController {

	@RequestMapping("/helloWorld")
	public String helloWorld(Model model) {
		model.addAttribute("message", "Hello World!");
		return "helloWorld";
	}
}
```

As you can see, the`@Controller`and`@RequestMapping`annotations allow flexible method names and signatures. In this particular example the method accepts a`Model`and returns a view name as a`String`, but various other method parameters and return values can be used as explained later in this section.`@Controller`and`@RequestMapping`and a number of other annotations form the basis for the Spring MVC implementation. This section documents these annotations and how they are most commonly used in a Servlet environment.

