### 18.10.4处理表单中的文件上传

在`MultipartResolver`完成其作业之后，请求将像其他处理一样进行处理。 首先，创建一个带有文件输入的表单，允许用户上传表单。 编码属性（`enctype =“multipart/form-data”`）让浏览器知道如何将表单编码为多部分请求：

```java
<html>
    <head>
        <title>Upload a file please</title>
    </head>
    <body>
        <h1>Please upload a file</h1>
        <form method="post" action="/form" enctype="multipart/form-data">
            <input type="text" name="name"/>
            <input type="file" name="file"/>
            <input type="submit"/>
        </form>
    </body>
</html>
```

下一步是创建一个处理文件上传的控制器。 除了我们在方法参数中使用`MultipartHttpServletRequest`或`MultipartFile`之外，这个控制器与[标准的`@Controller`](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-ann-controller)非常相似：

```java
@Controller
public class FileUploadController {

    @PostMapping("/form")
    public String handleFormUpload(@RequestParam("name") String name,
            @RequestParam("file") MultipartFile file) {

        if (!file.isEmpty()) {
            byte[] bytes = file.getBytes();
            // 将字节存储在某个地方
            return "redirect:uploadSuccess";
        }

        return "redirect:uploadFailure";
    }

}
```

请注意，`@RequestParam`方法参数如何映射到表单中声明的输入元素。 在这个例子中，`byte[]`没有做任何事情，但实际上你可以将它保存在数据库中，将它存储在文件系统中，等等。

使用Servlet 3.0多部分解析时，还可以使用`javax.servlet.http.Part`作为方法参数：

```java
@Controller
public class FileUploadController {

    @PostMapping("/form")
    public String handleFormUpload(@RequestParam("name") String name,
            @RequestParam("file") Part file) {

        InputStream inputStream = file.getInputStream();
        // 从上传的文件中存储字节
        return "redirect:uploadSuccess";
    }

}
```



