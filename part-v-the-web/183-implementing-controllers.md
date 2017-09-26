## 18.3实现 Controllers

Controllers 提供对通常通过服务接口定义的应用程序行为的访问。Controllers 解释用户输入并将其转换为由视图表示给用户的模型。 Spring以非常抽象的方式实现控制器，使您能够创建各种各样的控制器。

Spring 2.5引入了一种基于注释的编程模型，用于使用诸如`@RequestMapping`，`@RequestParam`，`@ModelAttribute`等注释的MVC控制器。以这种风格实现的控制器不必扩展特定的基类或实现特定的接口。此外，它们通常不直接依赖于Servlet API，但是如果需要，您可以轻松地配置对Servlet设施的访问。

| ![](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/tip.png) |
| :--- |
| 可用于[Github中的弹性项目Org](https://github.com/spring-projects/)，许多Web应用程序利用本节中描述的注释支持，包括_MvcShowcase_，_MvcAjax_，_MvcBasic_，_PetClinic_，_PetCare_等。 |

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

您可以看到，`@Controller`和`@RequestMapping`注释允许灵活的方法名称和签名。在这个特殊的例子中，该方法接受一个`Model`并返回一个视图名称作为一个`String`，但是可以使用各种其他的方法参数和返回值，如本节稍后所述。 `@Controller`和`@RequestMapping`和许多其他注释构成了Spring MVC实现的基础。本节介绍这些注释以及它们在Servlet环境中最常用的注释。



