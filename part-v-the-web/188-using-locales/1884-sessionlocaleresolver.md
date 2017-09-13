### 18.8.4 SessionLocaleResolver

The`SessionLocaleResolver`allows you to retrieve`Locale`and`TimeZone`from the session that might be associated with the user’s request. In contrast to`CookieLocaleResolver`, this strategy stores locally chosen locale settings in the Servlet container’s`HttpSession`. As a consequence, those settings are just temporary for each session and therefore lost when each session terminates.

Note that there is no direct relationship with external session management mechanisms such as the Spring Session project. This`SessionLocaleResolver`will simply evaluate and modify corresponding`HttpSession`attributes against the current`HttpServletRequest`.

