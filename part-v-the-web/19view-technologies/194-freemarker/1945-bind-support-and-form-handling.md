### 19.4.5 Bind support and form handling

Spring provides a tag library for use in JSP’s that contains \(amongst other things\) a`<spring:bind/>`tag. This tag primarily enables forms to display values from form backing objects and to show the results of failed validations from a`Validator`in the web or business tier. Spring also has support for the same functionality in FreeMarker, with additional convenience macros for generating form input elements themselves.

#### The bind macros

A standard set of macros are maintained within the`spring-webmvc.jar`file for both languages, so they are always available to a suitably configured application.

Some of the macros defined in the Spring libraries are considered internal \(private\) but no such scoping exists in the macro definitions making all macros visible to calling code and user templates. The following sections concentrate only on the macros you need to be directly calling from within your templates. If you wish to view the macro code directly, the file is called`spring.ftl`in the package`org.springframework.web.servlet.view.freemarker`.

#### Simple binding

In your HTML forms \(vm / ftl templates\) which act as a form view for a Spring MVC controller, you can use code similar to the following to bind to field values and display error messages for each input field in similar fashion to the JSP equivalent. Example code is shown below for the`personForm`view configured earlier:

```java
<!-- freemarker macros have to be imported into a namespace. We strongly
recommend sticking to 'spring' -->
<#import "/spring.ftl" as spring/>
<html>
	...
	<form action="" method="POST">
		Name:
		<@spring.bind "myModelObject.name"/>
		<input type="text"
			name="${spring.status.expression}"
			value="${spring.status.value?html}"/><br>
		<#list spring.status.errorMessages as error> <b>${error}</b> <br> </#list>
		<br>
		...
		<input type="submit" value="submit"/>
	</form>
	...
</html>
```

`<@spring.bind>`requires a 'path' argument which consists of the name of your command object \(it will be 'command' unless you changed it in your FormController properties\) followed by a period and the name of the field on the command object you wish to bind to. Nested fields can be used too such as "command.address.street". The`bind`macro assumes the default HTML escaping behavior specified by the ServletContext parameter`defaultHtmlEscape`in`web.xml`.

The optional form of the macro called`<@spring.bindEscaped>`takes a second argument and explicitly specifies whether HTML escaping should be used in the status error messages or values. Set to true or false as required. Additional form handling macros simplify the use of HTML escaping and these macros should be used wherever possible. They are explained in the next section.

#### Form input generation macros

Additional convenience macros for both languages simplify both binding and form generation \(including validation error display\). It is never necessary to use these macros to generate form input fields, and they can be mixed and matched with simple HTML or calls direct to the spring bind macros highlighted previously.

The following table of available macros show the VTL and FTL definitions and the parameter list that each takes.

**Table19.1.Table of macro definitions**

| macro | FTL definition | **message** | \(output a string from a resource bundle based on the code parameter\) |
| :--- | :--- | :--- | :--- |
|  | &lt;@spring.message code/&gt; | **messageText**\(output a string from a resource bundle based on the code parameter, falling back to the value of the default parameter\) | &lt;@spring.messageText code, text/&gt; |
|  | **url**\(prefix a relative URL with the application’s context root\) | &lt;@spring.url relativeUrl/&gt; | **formInput**\(standard input field for gathering user input\) |
|  | &lt;@spring.formInput path, attributes, fieldType/&gt; | **formHiddenInput\***\(hidden input field for submitting non-user input\) | &lt;@spring.formHiddenInput path, attributes/&gt; |
|  | **formPasswordInput**\* \(standard input field for gathering passwords. Note that no value will ever be populated in fields of this type\) | &lt;@spring.formPasswordInput path, attributes/&gt; | **formTextarea**\(large text field for gathering long, freeform text input\) |
|  | &lt;@spring.formTextarea path, attributes/&gt; | **formSingleSelect**\(drop down box of options allowing a single required value to be selected\) | &lt;@spring.formSingleSelect path, options, attributes/&gt; |
|  | **formMultiSelect**\(a list box of options allowing the user to select 0 or more values\) | &lt;@spring.formMultiSelect path, options, attributes/&gt; | **formRadioButtons**\(a set of radio buttons allowing a single selection to be made from the available choices\) |
|  | &lt;@spring.formRadioButtons path, options separator, attributes/&gt; | **formCheckboxes**\(a set of checkboxes allowing 0 or more values to be selected\) | &lt;@spring.formCheckboxes path, options, separator, attributes/&gt; |
|  | **formCheckbox**\(a single checkbox\) | &lt;@spring.formCheckbox path, attributes/&gt; | **showErrors**\(simplify display of validation errors for the bound field\) |

* In FTL \(FreeMarker\), these two macros are not actually required as you can use the normal`formInput`macro, specifying '`hidden’ or '`password’ as the value for the`fieldType`parameter.

The parameters to any of the above macros have consistent meanings:

* path: the name of the field to bind to \(ie "command.name"\)

* options: a Map of all the available values that can be selected from in the input field. The keys to the map represent the values that will be POSTed back from the form and bound to the command object. Map objects stored against the keys are the labels displayed on the form to the user and may be different from the corresponding values posted back by the form. Usually such a map is supplied as reference data by the controller. Any Map implementation can be used depending on required behavior. For strictly sorted maps, a`SortedMap`such as a`TreeMap`with a suitable Comparator may be used and for arbitrary Maps that should return values in insertion order, use a`LinkedHashMap`or a`LinkedMap`from commons-collections.

* separator: where multiple options are available as discreet elements \(radio buttons or checkboxes\), the sequence of characters used to separate each one in the list \(ie "&lt;br&gt;"\).

* attributes: an additional string of arbitrary tags or text to be included within the HTML tag itself. This string is echoed literally by the macro. For example, in a textarea field you may supply attributes as 'rows="5" cols="60"' or you could pass style information such as 'style="border:1px solid silver"'.

* classOrStyle: for the showErrors macro, the name of the CSS class that the span tag wrapping each error will use. If no information is supplied \(or the value is empty\) then the errors will be wrapped in&lt;b&gt;&lt;/b&gt;tags.

Examples of the macros are outlined below some in FTL and some in VTL. Where usage differences exist between the two languages, they are explained in the notes.

##### Input Fields

The formInput macro takes the path parameter \(command.name\) and an additional attributes parameter which is empty in the example above. The macro, along with all other form generation macros, performs an implicit spring bind on the path parameter. The binding remains valid until a new bind occurs so the showErrors macro doesn’t need to pass the path parameter again - it simply operates on whichever field a bind was last created for.

The showErrors macro takes a separator parameter \(the characters that will be used to separate multiple errors on a given field\) and also accepts a second parameter, this time a class name or style attribute. Note that FreeMarker is able to specify default values for the attributes parameter.

```java
<@spring.formInput "command.name"/>
<@spring.showErrors "<br>"/>
```

Output is shown below of the form fragment generating the name field, and displaying a validation error after the form was submitted with no value in the field. Validation occurs through Spring’s Validation framework.

The generated HTML looks like this:

```java
Name:
<input type="text" name="name" value="">
<br>
	<b>required</b>
<br>
<br>
```

The formTextarea macro works the same way as the formInput macro and accepts the same parameter list. Commonly, the second parameter \(attributes\) will be used to pass style information or rows and cols attributes for the textarea.

##### Selection Fields

Four selection field macros can be used to generate common UI value selection inputs in your HTML forms.

* formSingleSelect

* formMultiSelect

* formRadioButtons

* formCheckboxes

Each of the four macros accepts a Map of options containing the value for the form field, and the label corresponding to that value. The value and the label can be the same.

An example of radio buttons in FTL is below. The form backing object specifies a default value of 'London' for this field and so no validation is necessary. When the form is rendered, the entire list of cities to choose from is supplied as reference data in the model under the name 'cityMap'.

```java
...
Town:
<@spring.formRadioButtons "command.address.town", cityMap, ""/><br><br>
```

This renders a line of radio buttons, one for each value in`cityMap`using the separator "". No additional attributes are supplied \(the last parameter to the macro is missing\). The cityMap uses the same String for each key-value pair in the map. The map’s keys are what the form actually submits as POSTed request parameters, map values are the labels that the user sees. In the example above, given a list of three well known cities and a default value in the form backing object, the HTML would be

```js
Town:
<input type="radio" name="address.town" value="London">London</input>
<input type="radio" name="address.town" value="Paris" checked="checked">Paris</input>
<input type="radio" name="address.town" value="New York">New York</input>
```

If your application expects to handle cities by internal codes for example, the map of codes would be created with suitable keys like the example below.

```java
protected Map<String, String> referenceData(HttpServletRequest request) throws Exception {
	Map<String, String> cityMap = new LinkedHashMap<>();
	cityMap.put("LDN", "London");
	cityMap.put("PRS", "Paris");
	cityMap.put("NYC", "New York");

	Map<String, String> model = new HashMap<>();
	model.put("cityMap", cityMap);
	return model;
}
```

The code would now produce output where the radio values are the relevant codes but the user still sees the more user friendly city names.

```js
Town:
<input type="radio" name="address.town" value="LDN">London</input>
<input type="radio" name="address.town" value="PRS" checked="checked">Paris</input>
<input type="radio" name="address.town" value="NYC">New York</input>
```

#### HTML escaping and XHTML compliance

Default usage of the form macros above will result in HTML tags that are HTML 4.01 compliant and that use the default value for HTML escaping defined in your web.xml as used by Spring’s bind support. In order to make the tags XHTML compliant or to override the default HTML escaping value, you can specify two variables in your template \(or in your model where they will be visible to your templates\). The advantage of specifying them in the templates is that they can be changed to different values later in the template processing to provide different behavior for different fields in your form.

To switch to XHTML compliance for your tags, specify a value of 'true' for a model/context variable named xhtmlCompliant:

```java
<#-- for FreeMarker -->
<#assign xhtmlCompliant = true in spring>
```

Any tags generated by the Spring macros will now be XHTML compliant after processing this directive.

In similar fashion, HTML escaping can be specified per field:

```java
<#-- until this point, default HTML escaping is used -->

<#assign htmlEscape = true in spring>
<#-- next field will use HTML escaping -->
<@spring.formInput "command.name"/>

<#assign htmlEscape = false in spring>
<#-- all future fields will be bound with HTML escaping off -->
```



