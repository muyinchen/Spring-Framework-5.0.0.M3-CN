### 18.13.3View - RequestToViewNameTranslator

`RequestToViewNameTranslator`接口在没有明确提供这样的逻辑`View`名称时确定一个逻辑视图名称。 它只有一个实现，即`DefaultRequestToViewNameTranslator`类。

`DefaultRequestToViewNameTranslator`将请求URL映射到逻辑视图名称，如下例所示：

```java
public class RegistrationController implements Controller {

    public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) {
        // 处理请求...
        ModelAndView mav = new ModelAndView();
        // 根据需要添加数据到模型...
        return mav;
        // 注意没有设置View或逻辑视图名称
    }

}
```

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- 这个具有众所周知的名称的bean为我们生成视图名称 -->
    <bean id="viewNameTranslator"
            class="org.springframework.web.servlet.view.DefaultRequestToViewNameTranslator"/>

    <bean class="x.y.RegistrationController">
        <!-- 根据需要注入依赖项 -->
    </bean>

    <!-- 将请求URL映射到控制器名称 -->
    <bean class="org.springframework.web.servlet.mvc.support.ControllerClassNameHandlerMapping"/>

    <bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/jsp/"/>
        <property name="suffix" value=".jsp"/>
    </bean>
```

请注意，在`handleRequest(..)`方法的实现中，返回的`ModelAndView`中没有设置`View`或逻辑视图名称。 `DefaultRequestToViewNameTranslator`的任务是从请求的URL生成_逻辑视图名称_。 在与`ControllerClassNameHandlerMapping`一起使用的上述`RegistrationController`的情况下，`http://localhost/registration.html`的请求URL导致由`DefaultRequestToViewNameTranslator`生成的注册的逻辑视图名称。 这个逻辑视图名称然后被`InternalResourceViewResolver `bean解析到`/WEB-INF/jsp/registration.jsp`视图中。

| ![](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/tip.png) |
| :--- |
| 您不需要明确定义`DefaultRequestToViewNameTranslator` bean。 如果你喜欢`DefaultRequestToViewNameTranslator`的默认设置，你可以依靠Spring Web MVC `DispatcherServlet`实例化这个类的实例（如果没有明确配置的话）。. |

当然，如果您需要更改默认设置，则需要明确配置您自己的`DefaultRequestToViewNameTranslator` bean。 请查阅全面的`DefaultRequestToViewNameTranslator` javadocs，了解有关可配置的各种属性的详细信息。

