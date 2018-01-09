## 18.14 HTTP缓存支持

一个好的HTTP缓存策略可以显着提高Web应用程序的性能和客户的体验。 `'Cache-Control'`HTTP响应标头主要负责这一点，以及诸如`'Last-Modified'`和`'ETag'`的条件标头。

`'Cache-Control'`HTTP响应标头建议专用缓存（例如浏览器）和公共缓存（例如代理）如何缓存HTTP响应以供进一步重用。

[ETag](https://en.wikipedia.org/wiki/HTTP_ETag)（实体标签）是HTTP/1.1兼容的Web服务器返回的HTTP响应头，用于确定给定URL的内容更改。 它可以被认为是`Last-Modified`头的更复杂的后继者。 当服务器返回带有ETag头的表示时，客户端可以在随后的GET中，在`If-None-Match`头中使用该头。 如果内容没有改变，则服务器返回`304: Not Modified`。

本节描述了在Spring Web MVC应用程序中配置HTTP缓存的不同选择。

