### 18.10.1 介绍

Spring的内置多部分支持处理Web应用程序中的文件上传。 您可以使用`org.springframework.web.multipart`包中定义的可插入`MultipartResolver`对象来启用此多部分支持。 Spring提供了一个`MultipartResolver`实现，用于与[_Commons FileUpload的实现_](https://jakarta.apache.org/commons/fileupload)一起使用，另一个用于Servlet 3.0多部分请求分析。

默认情况下，Spring没有多部分处理，因为有些开发人员想要自己处理多部分。 通过将多部分解析器添加到Web应用程序的上下文中来启用Spring多部分处理。 检查每个请求以查看它是否包含多部分。 如果找不到多部分，请求将按预期继续。 如果在请求中找到multipart，则使用在上下文中声明的`MultipartResolver`。 之后，请求中的多部分属性就像任何其他属性一样处理。

###  



