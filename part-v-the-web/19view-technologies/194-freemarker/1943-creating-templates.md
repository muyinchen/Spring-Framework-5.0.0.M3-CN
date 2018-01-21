### 19.4.3 Creating templates

Your templates need to be stored in the directory specified by the`FreeMarkerConfigurer`shown above. If you use the view resolvers highlighted, then the logical view names relate to the template file names in similar fashion to`InternalResourceViewResolver`for JSP’s. So if your controller returns a ModelAndView object containing a view name of "welcome" then the resolver will look for the`/WEB-INF/freemarker/welcome.ftl`template.

