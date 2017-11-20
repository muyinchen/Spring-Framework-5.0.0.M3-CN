### 18.5.2链接视图解析器

Spring支持多个视图解析器。 因此，您可以链接解析器，并在某些情况下覆盖特定的视图。 您可以通过向应用程序上下文中添加多个解析器来链接视图解析器，并在必要时通过设置`order`属性来指定排序。 请记住，订单属性越高，视图解析器在链中的位置越晚。

在以下示例中，视图解析器链由两个解析器组成：一个`InternalResourceViewResolver`（始终自动定位为链中的最后一个解析器）以及一个用于指定Excel视图的`XmlViewResolver`。 Excel视图不受`InternalResourceViewResolver`支持。

```java
<bean id="jspViewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="viewClass" value="org.springframework.web.servlet.view.JstlView"/>
    <property name="prefix" value="/WEB-INF/jsp/"/>
    <property name="suffix" value=".jsp"/>
</bean>

<bean id="excelViewResolver" class="org.springframework.web.servlet.view.XmlViewResolver">
    <property name="order" value="1"/>
    <property name="location" value="/WEB-INF/views.xml"/>
</bean>

<!-- in views.xml -->

<beans>
    <bean name="report" class="org.springframework.example.ReportExcelView"/>
</beans>
```

如果一个特定的视图解析器没有产生视图，Spring会检查其他视图解析器的上下文。 如果存在额外的视图解析器，Spring将继续检查它们，直到视图解决。 如果没有视图解析器返回一个视图，Spring会抛出一个`ServletException`异常。

视图解析器的合约指定一个视图resolver\_can\_return为null以指示视图找不到。 然而并不是所有的视图解析器都这样做，因为在某些情况下，解析器根本无法检测视图是否存在。 例如，`InternalResourceViewResolver`在内部使用`RequestDispatcher`，调度是确定JSP是否存在的唯一方法，但此操作只能执行一次。 `FreeMarkerViewResolver`和其他的一样。 检查特定视图解析器的javadoc以查看是否报告不存在的视图。 因此，将一个`InternalResourceViewResolver`放在链中的最后一个结果链中没有被完全检查，因为`InternalResourceViewResolver `will\_always\_return一个视图！

