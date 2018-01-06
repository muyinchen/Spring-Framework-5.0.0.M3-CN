### 18.10.5处理来自编程客户端的文件上传请求

也可以从RESTful服务场景中的非浏览器客户端提交多部分请求。 以上所有示例和配置都适用于此。 但是，与通常提交文件和简单表单域的浏览器不同，编程客户机还可以发送更复杂的特定内容类型的数据 - 例如带文件的多部分请求，带JSON格式数据的第二部分：

```java
POST /someUrl
Content-Type: multipart/mixed

--edt7Tfrdusa7r3lNQc79vXuhIIMlatb7PQg7Vp
Content-Disposition: form-data; name="meta-data"
Content-Type: application/json; charset=UTF-8
Content-Transfer-Encoding: 8bit

{
    "name": "value"
}
--edt7Tfrdusa7r3lNQc79vXuhIIMlatb7PQg7Vp
Content-Disposition: form-data; name="file-data"; filename="file.properties"
Content-Type: text/xml
Content-Transfer-Encoding: 8bit
... File Data ...
```

You could access the part named "meta-data" with a`@RequestParam("meta-data") String metadata`controller method argument. However, you would probably prefer to accept a strongly typed object initialized from the JSON formatted data in the body of the request part, very similar to the way`@RequestBody`converts the body of a non-multipart request to a target object with the help of an`HttpMessageConverter`.

You can use the`@RequestPart`annotation instead of the`@RequestParam`annotation for this purpose. It allows you to have the content of a specific multipart passed through an`HttpMessageConverter`taking into consideration the`'Content-Type'`header of the multipart:

您可以使用`@RequestParam("meta-data") String metadata`控制器方法参数来访问名为"meta-data" 的部分。 但是，您可能更愿意接受一个从请求部分主体中的JSON格式的数据初始化的强类型对象，这与`@RequestBody`在非帮助的情况下将非多部分请求的主体转换为目标对象的方式非常相似 一个`HttpMessageConverter`。

您可以使用`@RequestPart`注解而不是`@RequestParam`注解来达到此目的。 它允许您通过`HttpMessageConverter`考虑多部分的`'Content-Type'`标题来传递特定多部分的内容：

```java
@PostMapping("/someUrl")
public String onSubmit(@RequestPart("meta-data") MetaData metadata,
        @RequestPart("file-data") MultipartFile file) {

    // ...

}
```

请注意，`MultipartFile`方法参数如何通过`@RequestParam`或`@RequestPart`可互换地访问。 然而，在这种情况下，`@RequestPart("meta-data") MetaData`方法参数被读取为基于其`'Content-Type'`头的JSON内容，并且借助于`MappingJackson2HttpMessageConverter`进行转换。

