### 19.9.1My First Words

This example is a trivial Spring application that creates a list of words in the`Controller`and adds them to the model map. The map is returned along with the view name of our XSLT view. See[Section18.3, “Implementing Controllers”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-controller)for details of Spring Web MVC’s`Controller`interface. The XSLT Controller will turn the list of words into a simple XML document ready for transformation.

#### Bean definitions

Configuration is standard for a simple Spring application. The MVC configuration has to define a`XsltViewResolver`bean and regular MVC annotation configuration.

```java
@EnableWebMvc
@ComponentScan
@Configuration
public class WebConfig extends WebMvcConfigurerAdapter {

	@Bean
	public XsltViewResolver xsltViewResolver() {
		XsltViewResolver viewResolver = new XsltViewResolver();
		viewResolver.setPrefix("/WEB-INF/xsl/");
		viewResolver.setSuffix(".xslt");
		return viewResolver;
	}

}
```

And we need a Controller that encapsulates our word generation logic.

#### Standard MVC controller code

The controller logic is encapsulated in a`@Controller`class, with the handler method being defined like so…​

```java
@Controller
public class XsltController {

	@RequestMapping("/")
	public String home(Model model) throws Exception {

		Document document = DocumentBuilderFactory.newInstance().newDocumentBuilder().newDocument();
		Element root = document.createElement("wordList");

		List<String> words = Arrays.asList("Hello", "Spring", "Framework");
		for (String word : words) {
			Element wordNode = document.createElement("word");
			Text textNode = document.createTextNode(word);
			wordNode.appendChild(textNode);
			root.appendChild(wordNode);
		}

		model.addAttribute("wordList", root);
		return "home";
	}
```

So far we’ve only created a DOM document and added it to the Model map. Note that you can also load an XML file as a`Resource`and use it instead of a custom DOM document.

Of course, there are software packages available that will automatically 'domify' an object graph, but within Spring, you have complete flexibility to create the DOM from your model in any way you choose. This prevents the transformation of XML playing too great a part in the structure of your model data which is a danger when using tools to manage the domification process.

Next,`XsltViewResolver`will resolve the "home" XSLT template file and merge the DOM document into it to generate our view.

#### Document transformation

Finally, the`XsltViewResolver`will resolve the "home" XSLT template file and merge the DOM document into it to generate our view. As shown in the`XsltViewResolver`configuration, XSLT templates live in the war file in the`'WEB-INF/xsl'`directory and end with a`"xslt"`file extension.

```js

<?xml version="1.0" encoding="utf-8"?>
<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">

	<xsl:output method="html" omit-xml-declaration="yes"/>

	<xsl:template match="/">
		<html>
			<head><title>Hello!</title></head>
			<body>
				<h1>My First Words</h1>
				<ul>
					<xsl:apply-templates/>
				</ul>
			</body>
		</html>
	</xsl:template>

	<xsl:template match="word">
		<li><xsl:value-of select="."/></li>
	</xsl:template>

</xsl:stylesheet>
```

This is rendered as:

```js
<html>
	<head>
		<META http-equiv="Content-Type" content="text/html; charset=UTF-8">
		<title>Hello!</title>
	</head>
	<body>
		<h1>My First Words</h1>
		<ul>
			<li>Hello</li>
			<li>Spring</li>
			<li>Framework</li>
		</ul>
	</body>
</html>
```



