### 18.10.5Handling a file upload request from programmatic clients

Multipart requests can also be submitted from non-browser clients in a RESTful service scenario. All of the above examples and configuration apply here as well. However, unlike browsers that typically submit files and simple form fields, a programmatic client can also send more complex data of a specific content type — for example a multipart request with a file and second part with JSON formatted data:

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

```java
@PostMapping("/someUrl")
public String onSubmit(@RequestPart("meta-data") MetaData metadata,
		@RequestPart("file-data") MultipartFile file) {

	// ...

}
```

Notice how`MultipartFile`method arguments can be accessed with`@RequestParam`or with`@RequestPart`interchangeably. However, the`@RequestPart("meta-data") MetaData`method argument in this case is read as JSON content based on its`'Content-Type'`header and converted with the help of the`MappingJackson2HttpMessageConverter`.

