### 18.8.1 获取时区信息

除了获取客户端的语言环境之外，了解他们的时区通常也很有用。 `LocaleContextResolver`接口为`LocaleResolver`提供了一个扩展，它允许解析器提供更丰富的`LocaleContext`，其中可能包含时区信息。

如果可用，用户的`TimeZone`可以使用`RequestContext.getTimeZone()`方法获得。 时区信息将自动被Spring的`ConversionService`注册的日期/时间`Converter`和`Formatter`对象使用。

