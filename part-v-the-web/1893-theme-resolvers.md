### 18.9.3Theme resolvers

After you define themes, as in the preceding section, you decide which theme to use. The`DispatcherServlet`will look for a bean named`themeResolver`to find out which`ThemeResolver`implementation to use. A theme resolver works in much the same way as a`LocaleResolver`. It detects the theme to use for a particular request and can also alter the request’s theme. The following theme resolvers are provided by Spring:

**Table18.5.ThemeResolver implementations**

| Class | Description |
| :--- | :--- |
| `FixedThemeResolver` | Selects a fixed theme, set using the`defaultThemeName`property. |
| `SessionThemeResolver` | The theme is maintained in the user’s HTTP session. It only needs to be set once for each session, but is not persisted between sessions. |
| `CookieThemeResolver` | The selected theme is stored in a cookie on the client. |

Spring also provides a`ThemeChangeInterceptor`that allows theme changes on every request with a simple request parameter.

