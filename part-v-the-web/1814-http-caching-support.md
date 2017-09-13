## 18.14 HTTP caching support

A good HTTP caching strategy can significantly improve the performance of a web application and the experience of its clients. The`'Cache-Control'`HTTP response header is mostly responsible for this, along with conditional headers such as`'Last-Modified'`and`'ETag'`.

The`'Cache-Control'`HTTP response header advises private caches \(e.g. browsers\) and public caches \(e.g. proxies\) on how they can cache HTTP responses for further reuse.

An[ETag](https://en.wikipedia.org/wiki/HTTP_ETag)\(entity tag\) is an HTTP response header returned by an HTTP/1.1 compliant web server used to determine change in content at a given URL. It can be considered to be the more sophisticated successor to the`Last-Modified`header. When a server returns a representation with an ETag header, the client can use this header in subsequent GETs, in an`If-None-Match`header. If the content has not changed, the server returns`304: Not Modified`.

This section describes the different choices available to configure HTTP caching in a Spring Web MVC application.

