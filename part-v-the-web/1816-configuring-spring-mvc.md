## 18.16 配置Spring MVC

[第18.2.1节“WebApplicationContext中的特殊Bean类型”](#)和[第18.2.2节“默认DispatcherServlet配置”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-servlet-config)介绍了Spring MVC的特殊bean以及`DispatcherServlet`使用的默认实现。 在本节中，您将学习到另外两种配置Spring MVC的方法。 即MVC Java配置和MVC XML命名空间。

MVC Java配置和MVC命名空间提供了类似的默认配置，覆盖了`DispatcherServlet`的默认设置。 我们的目标是让大多数应用程序不必创建相同的配置，也可以提供更高级别的构造，用于配置Spring MVC，作为一个简单的起点，并且不需要事先知道底层配置。

您可以根据您的偏好选择MVC Java配置或MVC命名空间。 同样如您将在下面看到的，使用MVC Java配置，可以更容易地看到底层配置以及直接对创建的Spring MVC bean进行细粒度的自定义。 但是我们从头开始。

