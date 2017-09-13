### 18.8.1 Obtaining Time Zone Information

In addition to obtaining the client’s locale, it is often useful to know their time zone. The`LocaleContextResolver`interface offers an extension to`LocaleResolver`that allows resolvers to provide a richer`LocaleContext`, which may include time zone information.

When available, the user’s`TimeZone`can be obtained using the`RequestContext.getTimeZone()`method. Time zone information will automatically be used by Date/Time`Converter`and`Formatter`objects registered with Spring’s`ConversionService`.

