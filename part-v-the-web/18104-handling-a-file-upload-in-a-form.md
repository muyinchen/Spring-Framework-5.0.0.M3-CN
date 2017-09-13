### 18.10.4Handling a file upload in a form

After the`MultipartResolver`completes its job, the request is processed like any other. First, create a form with a file input that will allow the user to upload a form. The encoding attribute \(`enctype="multipart/form-data"`\) lets the browser know how to encode the form as multipart request:

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

The next step is to create a controller that handles the file upload. This controller is very similar to a[normal annotated`@Controller`](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-ann-controller), except that we use`MultipartHttpServletRequest`or`MultipartFile`in the method parameters:

```java
@Controller
public class FileUploadController {

	@PostMapping("/form")
	public String handleFormUpload(@RequestParam("name") String name,
			@RequestParam("file") MultipartFile file) {

		if (!file.isEmpty()) {
			byte[] bytes = file.getBytes();
			// store the bytes somewhere
			return "redirect:uploadSuccess";
		}

		return "redirect:uploadFailure";
	}

}
```

Note how the`@RequestParam`method parameters map to the input elements declared in the form. In this example, nothing is done with the`byte[]`, but in practice you can save it in a database, store it on the file system, and so on.

When using Servlet 3.0 multipart parsing you can also use`javax.servlet.http.Part`for the method parameter:

```java
@Controller
public class FileUploadController {

	@PostMapping("/form")
	public String handleFormUpload(@RequestParam("name") String name,
			@RequestParam("file") Part file) {

		InputStream inputStream = file.getInputStream();
		// store bytes from uploaded file somewhere

		return "redirect:uploadSuccess";
	}

}
```



