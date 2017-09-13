### 18.9.2Defining themes

To use themes in your web application, you must set up an implementation of the`org.springframework.ui.context.ThemeSource`interface. The`WebApplicationContext`interface extends`ThemeSource`but delegates its responsibilities to a dedicated implementation. By default the delegate will be an`org.springframework.ui.context.support.ResourceBundleThemeSource`implementation that loads properties files from the root of the classpath. To use a custom`ThemeSource`implementation or to configure the base name prefix of the`ResourceBundleThemeSource`, you can register a bean in the application context with the reserved name`themeSource`. The web application context automatically detects a bean with that name and uses it.

When using the`ResourceBundleThemeSource`, a theme is defined in a simple properties file. The properties file lists the resources that make up the theme. Here is an example:

```java
styleSheet=/themes/cool/style.css
background=/themes/cool/img/coolBg.jpg
```

The keys of the properties are the names that refer to the themed elements from view code. For a JSP, you typically do this using the`spring:theme`custom tag, which is very similar to the`spring:message`tag. The following JSP fragment uses the theme defined in the previous example to customize the look and feel:

```java
<%@ taglib prefix="spring" uri="http://www.springframework.org/tags"%>
<html>
	<head>
		<link rel="stylesheet" href="<spring:theme code='styleSheet'/>" type="text/css"/>
	</head>
	<body style="background=<spring:theme code='background'/>">
		...
	</body>
</html>
```

By default, the`ResourceBundleThemeSource`uses an empty base name prefix. As a result, the properties files are loaded from the root of the classpath. Thus you would put the`cool.properties`theme definition in a directory at the root of the classpath, for example, in`/WEB-INF/classes`. The`ResourceBundleThemeSource`uses the standard Java resource bundle loading mechanism, allowing for full internationalization of themes. For example, we could have a`/WEB-INF/classes/cool_nl.properties`that references a special background image with Dutch text on it.

