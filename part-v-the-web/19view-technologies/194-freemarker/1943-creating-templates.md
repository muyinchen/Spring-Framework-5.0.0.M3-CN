### 19.4.3 创建模板

您的模板需要存储在上面显示的`FreeMarkerConfigurer`指定的目录中。 如果使用突出显示的视图解析器，那么逻辑视图名称与模板文件名称的关联类似于用于JSP的`InternalResourceViewResolver`。 因此，如果您的控制器返回一个包含“welcome”视图名称的ModelAndView对象，则解析器将查找`/WEB-INF/freemarker/welcome.ftl`模板。

