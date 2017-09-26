### 18.3.1使用@Controller定义控制器

`@Controller`注释表示特定的类用于控制器的角色。 Spring不需要扩展任何控制器基类或引用Servlet API。 但是，如果需要，您仍然可以参考Servlet特定的功能。

`@Controller`注释作为注释类的构造型，表示其作用。 调度程序扫描这些注释类的映射方法，并检测`@RequestMapping`注释（请参阅下一节）。

您可以使用调度程序上下文中的标准Spring bean定义来明确定义带注释的控制器bean。 但是，`@Controller`构造型还允许自动检测，与Spring通用支持对齐，用于检测类路径中的组件类并自动注册它们的bean定义。要启用自动检测这些带注释的控制器，您可以向组态添加组件扫描。 使用spring-context模式，如以下XML代码片段所示：

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="org.springframework.samples.petclinic.web"/>

    <!-- ... -->

</beans>
```



