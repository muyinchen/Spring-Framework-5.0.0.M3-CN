### 18.5.2Chaining ViewResolvers

Spring supports multiple view resolvers. Thus you can chain resolvers and, for example, override specific views in certain circumstances. You chain view resolvers by adding more than one resolver to your application context and, if necessary, by setting the`order`property to specify ordering. Remember, the higher the order property, the later the view resolver is positioned in the chain.

In the following example, the chain of view resolvers consists of two resolvers, an`InternalResourceViewResolver`, which is always automatically positioned as the last resolver in the chain, and an`XmlViewResolver`for specifying Excel views. Excel views are not supported by the`InternalResourceViewResolver`.

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

If a specific view resolver does not result in a view, Spring examines the context for other view resolvers. If additional view resolvers exist, Spring continues to inspect them until a view is resolved. If no view resolver returns a view, Spring throws a`ServletException`.

The contract of a view resolver specifies that a view resolver_can_return null to indicate the view could not be found. Not all view resolvers do this, however, because in some cases, the resolver simply cannot detect whether or not the view exists. For example, the`InternalResourceViewResolver`uses the`RequestDispatcher`internally, and dispatching is the only way to figure out if a JSP exists, but this action can only execute once. The same holds for the`FreeMarkerViewResolver`and some others. Check the javadocs of the specific view resolver to see whether it reports non-existing views. Thus, putting an`InternalResourceViewResolver`in the chain in a place other than the last results in the chain not being fully inspected, because the`InternalResourceViewResolver`will_always_return a view!

  


