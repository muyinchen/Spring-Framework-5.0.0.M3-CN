### 19.5.4 使用Spring的表单标签库

从版本2.0开始，Spring提供了一套全面的数据绑定感知标签，用于处理使用JSP和Spring Web MVC的表单元素。 每个标签都提供对其对应的HTML标签对应的一组属性的支持，使标签变得熟悉和直观地使用。 标签生成的HTML符合HTML 4.01 / XHTML 1.0。

与其他form / input标签库不同，Spring的form标签库与Spring Web MVC集成，使标签可以访问控制器处理的命令对象和引用数据。 正如您在下面的示例中所看到的，表单标签使得JSP更易于开发，读取和维护。

让我们通过表单标签，看看每个标签是如何使用的例子。 我们已经包括生成的HTML片段，其中某些标签需要进一步的评论。

#### 配置

表单标签库捆绑在`spring-webmvc.jar`中。 库描述符称为`spring-form.tld`。

要使用此库中的标签，请将以下指令添加到JSP页面的顶部：

```js
<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>
```

其中`form`是要用于来自此库的标记的标记名称前缀。

#### form标签

这个标签呈现一个HTML的'form'标签，并暴露一个绑定路径到内部标签进行绑定。 它将命令对象放在`PageContext`中，以便命令对象可以被内部标签访问。这个库中的所有其他标签都是标签的嵌套标签。

假设我们有一个名为`User`的域对象。 它是一个具有诸如`firstName`和`lastName`之类的属性的JavaBean。 我们将使用它作为我们的form控制器返回`form.jsp`的形式支持对象。 下面是`form.jsp`的一个例子：

```js
<form:form>
    <table>
        <tr>
            <td>First Name:</td>
            <td><form:input path="firstName"/></td>
        </tr>
        <tr>
            <td>Last Name:</td>
            <td><form:input path="lastName"/></td>
        </tr>
        <tr>
            <td colspan="2">
                <input type="submit" value="Save Changes"/>
            </td>
        </tr>
    </table>
</form:form>
```

The`firstName`and`lastName`values are retrieved from the command object placed in the`PageContext`by the page controller. Keep reading to see more complex examples of how inner tags are used with the`form`tag.

The generated HTML looks like a standard form:

```js
<form method="POST">
    <table>
        <tr>
            <td>First Name:</td>
            <td><input name="firstName" type="text" value="Harry"/></td>
        </tr>
        <tr>
            <td>Last Name:</td>
            <td><input name="lastName" type="text" value="Potter"/></td>
        </tr>
        <tr>
            <td colspan="2">
                <input type="submit" value="Save Changes"/>
            </td>
        </tr>
    </table>
</form>
```

The preceding JSP assumes that the variable name of the form backing object is`'command'`. If you have put the form backing object into the model under another name \(definitely a best practice\), then you can bind the form to the named variable like so:

```js
<form:form modelAttribute="user">
    <table>
        <tr>
            <td>First Name:</td>
            <td><form:input path="firstName"/></td>
        </tr>
        <tr>
            <td>Last Name:</td>
            <td><form:input path="lastName"/></td>
        </tr>
        <tr>
            <td colspan="2">
                <input type="submit" value="Save Changes"/>
            </td>
        </tr>
    </table>
</form:form>
```

#### The input tag

This tag renders an HTML 'input' tag using the bound value and type='text' by default. For an example of this tag, see[the section called “The form tag”](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/view.html#view-jsp-formtaglib-formtag). Starting with Spring 3.1 you can use other types such HTML5-specific types like 'email', 'tel', 'date', and others.

#### The checkbox tag

This tag renders an HTML 'input' tag with type 'checkbox'.

Let’s assume our`User`has preferences such as newsletter subscription and a list of hobbies. Below is an example of the`Preferences`class:

```java
public class Preferences {

    private boolean receiveNewsletter;
    private String[] interests;
    private String favouriteWord;

    public boolean isReceiveNewsletter() {
        return receiveNewsletter;
    }

    public void setReceiveNewsletter(boolean receiveNewsletter) {
        this.receiveNewsletter = receiveNewsletter;
    }

    public String[] getInterests() {
        return interests;
    }

    public void setInterests(String[] interests) {
        this.interests = interests;
    }

    public String getFavouriteWord() {
        return favouriteWord;
    }

    public void setFavouriteWord(String favouriteWord) {
        this.favouriteWord = favouriteWord;
    }
}
```

The`form.jsp`would look like:

```js
<form:form>
    <table>
        <tr>
            <td>Subscribe to newsletter?:</td>
            <%-- Approach 1: Property is of type java.lang.Boolean --%>
            <td><form:checkbox path="preferences.receiveNewsletter"/></td>
        </tr>

        <tr>
            <td>Interests:</td>
            <%-- Approach 2: Property is of an array or of type java.util.Collection --%>
            <td>
                Quidditch: <form:checkbox path="preferences.interests" value="Quidditch"/>
                Herbology: <form:checkbox path="preferences.interests" value="Herbology"/>
                Defence Against the Dark Arts: <form:checkbox path="preferences.interests" value="Defence Against the Dark Arts"/>
            </td>
        </tr>

        <tr>
            <td>Favourite Word:</td>
            <%-- Approach 3: Property is of type java.lang.Object --%>
            <td>
                Magic: <form:checkbox path="preferences.favouriteWord" value="Magic"/>
            </td>
        </tr>
    </table>
</form:form>
```

There are 3 approaches to the`checkbox`tag which should meet all your checkbox needs.

* Approach One - When the bound value is of type`java.lang.Boolean`, the`input(checkbox)`is marked as 'checked' if the bound value is`true`. The`value`attribute corresponds to the resolved value of the`setValue(Object)`value property.

* Approach Two - When the bound value is of type`array`or`java.util.Collection`, the`input(checkbox)`is marked as 'checked' if the configured`setValue(Object)`value is present in the bound`Collection`.

* Approach Three - For any other bound value type, the`input(checkbox)`is marked as 'checked' if the configured`setValue(Object)`is equal to the bound value.

Note that regardless of the approach, the same HTML structure is generated. Below is an HTML snippet of some checkboxes:

```js
<tr>
    <td>Interests:</td>
    <td>
        Quidditch: <input name="preferences.interests" type="checkbox" value="Quidditch"/>
        <input type="hidden" value="1" name="_preferences.interests"/>
        Herbology: <input name="preferences.interests" type="checkbox" value="Herbology"/>
        <input type="hidden" value="1" name="_preferences.interests"/>
        Defence Against the Dark Arts: <input name="preferences.interests" type="checkbox" value="Defence Against the Dark Arts"/>
        <input type="hidden" value="1" name="_preferences.interests"/>
    </td>
</tr>
```

What you might not expect to see is the additional hidden field after each checkbox. When a checkbox in an HTML page is_not\_checked, its value will not be sent to the server as part of the HTTP request parameters once the form is submitted, so we need a workaround for this quirk in HTML in order for Spring form data binding to work. The_`checkbox`_tag follows the existing Spring convention of including a hidden parameter prefixed by an underscore \("\_"\) for each checkbox. By doing this, you are effectively telling Spring that "_the checkbox was visible in the form and I want my object to which the form data will be bound to reflect the state of the checkbox no matter what_".

#### The checkboxes tag

This tag renders multiple HTML 'input' tags with type 'checkbox'.

Building on the example from the previous`checkbox`tag section. Sometimes you prefer not to have to list all the possible hobbies in your JSP page. You would rather provide a list at runtime of the available options and pass that in to the tag. That is the purpose of the`checkboxes`tag. You pass in an`Array`, a`List`or a`Map`containing the available options in the "items" property. Typically the bound property is a collection so it can hold multiple values selected by the user. Below is an example of the JSP using this tag:

```js
<form:form>
    <table>
        <tr>
            <td>Interests:</td>
            <td>
                <%-- Property is of an array or of type java.util.Collection --%>
                <form:checkboxes path="preferences.interests" items="${interestList}"/>
            </td>
        </tr>
    </table>
</form:form>
```

This example assumes that the "interestList" is a`List`available as a model attribute containing strings of the values to be selected from. In the case where you use a Map, the map entry key will be used as the value and the map entry’s value will be used as the label to be displayed. You can also use a custom object where you can provide the property names for the value using "itemValue" and the label using "itemLabel".

#### The radiobutton tag

This tag renders an HTML 'input' tag with type 'radio'.

A typical usage pattern will involve multiple tag instances bound to the same property but with different values.

```js
<tr>
    <td>Sex:</td>
    <td>
        Male: <form:radiobutton path="sex" value="M"/> <br/>
        Female: <form:radiobutton path="sex" value="F"/>
    </td>
</tr>
```

#### The radiobuttons tag

This tag renders multiple HTML 'input' tags with type 'radio'.

Just like the`checkboxes`tag above, you might want to pass in the available options as a runtime variable. For this usage you would use the`radiobuttons`tag. You pass in an`Array`, a`List`or a`Map`containing the available options in the "items" property. In the case where you use a Map, the map entry key will be used as the value and the map entry’s value will be used as the label to be displayed. You can also use a custom object where you can provide the property names for the value using "itemValue" and the label using "itemLabel".

```js
<tr>
    <td>Sex:</td>
    <td><form:radiobuttons path="sex" items="${sexOptions}"/></td>
</tr>
```

#### The password tag

This tag renders an HTML 'input' tag with type 'password' using the bound value.

```js
<tr>
    <td>Password:</td>
    <td>
        <form:password path="password"/>
    </td>
</tr>
```

Please note that by default, the password value is\_not\_shown. If you do want the password value to be shown, then set the value of the`'showPassword'`attribute to true, like so.

```js
<tr>
    <td>Password:</td>
    <td>
        <form:password path="password" value="^76525bvHGq" showPassword="true"/>
    </td>
</tr>
```

#### The select tag

This tag renders an HTML 'select' element. It supports data binding to the selected option as well as the use of nested`option`and`options`tags.

Let’s assume a`User`has a list of skills.

```js
<tr>
    <td>Skills:</td>
    <td><form:select path="skills" items="${skills}"/></td>
</tr>
```

If the`User’s`skill were in Herbology, the HTML source of the 'Skills' row would look like:

```js
<tr>
    <td>Skills:</td>
    <td>
        <select name="skills" multiple="true">
            <option value="Potions">Potions</option>
            <option value="Herbology" selected="selected">Herbology</option>
            <option value="Quidditch">Quidditch</option>
        </select>
    </td>
</tr>
```

#### The option tag

This tag renders an HTML 'option'. It sets 'selected' as appropriate based on the bound value.

```js
<tr>
    <td>House:</td>
    <td>
        <form:select path="house">
            <form:option value="Gryffindor"/>
            <form:option value="Hufflepuff"/>
            <form:option value="Ravenclaw"/>
            <form:option value="Slytherin"/>
        </form:select>
    </td>
</tr>
```

If the`User’s`house was in Gryffindor, the HTML source of the 'House' row would look like:

```js
<tr>
    <td>House:</td>
    <td>
        <select name="house">
            <option value="Gryffindor" selected="selected">Gryffindor</option>
            <option value="Hufflepuff">Hufflepuff</option>
            <option value="Ravenclaw">Ravenclaw</option>
            <option value="Slytherin">Slytherin</option>
        </select>
    </td>
</tr>
```

#### The options tag

This tag renders a list of HTML 'option' tags. It sets the 'selected' attribute as appropriate based on the bound value.

```js
<tr>
    <td>Country:</td>
    <td>
        <form:select path="country">
            <form:option value="-" label="--Please Select"/>
            <form:options items="${countryList}" itemValue="code" itemLabel="name"/>
        </form:select>
    </td>
</tr>
```

If the`User`lived in the UK, the HTML source of the 'Country' row would look like:

```js
<tr>
    <td>Country:</td>
    <td>
        <select name="country">
            <option value="-">--Please Select</option>
            <option value="AT">Austria</option>
            <option value="UK" selected="selected">United Kingdom</option>
            <option value="US">United States</option>
        </select>
    </td>
</tr>
```

As the example shows, the combined usage of an`option`tag with the`options`tag generates the same standard HTML, but allows you to explicitly specify a value in the JSP that is for display only \(where it belongs\) such as the default string in the example: "-- Please Select".

The`items`attribute is typically populated with a collection or array of item objects.`itemValue`and`itemLabel`simply refer to bean properties of those item objects, if specified; otherwise, the item objects themselves will be stringified. Alternatively, you may specify a`Map`of items, in which case the map keys are interpreted as option values and the map values correspond to option labels. If`itemValue`and/or`itemLabel`happen to be specified as well, the item value property will apply to the map key and the item label property will apply to the map value.

#### The textarea tag

This tag renders an HTML 'textarea'.

```js
<tr>
    <td>Notes:</td>
    <td><form:textarea path="notes" rows="3" cols="20"/></td>
    <td><form:errors path="notes"/></td>
</tr>
```

#### The hidden tag

This tag renders an HTML 'input' tag with type 'hidden' using the bound value. To submit an unbound hidden value, use the HTML`input`tag with type 'hidden'.

```js
<form:hidden path="house"/>
```

If we choose to submit the 'house' value as a hidden one, the HTML would look like:

```js
<input name="house" type="hidden" value="Gryffindor"/>
```

#### The errors tag

This tag renders field errors in an HTML 'span' tag. It provides access to the errors created in your controller or those that were created by any validators associated with your controller.

Let’s assume we want to display all error messages for the`firstName`and`lastName`fields once we submit the form. We have a validator for instances of the`User`class called`UserValidator`.

```java
public class UserValidator implements Validator {

    public boolean supports(Class candidate) {
        return User.class.isAssignableFrom(candidate);
    }

    public void validate(Object obj, Errors errors) {
        ValidationUtils.rejectIfEmptyOrWhitespace(errors, "firstName", "required", "Field is required.");
        ValidationUtils.rejectIfEmptyOrWhitespace(errors, "lastName", "required", "Field is required.");
    }
}
```

The`form.jsp`would look like:

```js
<form:form>
    <table>
        <tr>
            <td>First Name:</td>
            <td><form:input path="firstName"/></td>
            <%-- Show errors for firstName field --%>
            <td><form:errors path="firstName"/></td>
        </tr>

        <tr>
            <td>Last Name:</td>
            <td><form:input path="lastName"/></td>
            <%-- Show errors for lastName field --%>
            <td><form:errors path="lastName"/></td>
        </tr>
        <tr>
            <td colspan="3">
                <input type="submit" value="Save Changes"/>
            </td>
        </tr>
    </table>
</form:form>
```

If we submit a form with empty values in the`firstName`and`lastName`fields, this is what the HTML would look like:

```js
<form method="POST">
    <table>
        <tr>
            <td>First Name:</td>
            <td><input name="firstName" type="text" value=""/></td>
            <%-- Associated errors to firstName field displayed --%>
            <td><span name="firstName.errors">Field is required.</span></td>
        </tr>

        <tr>
            <td>Last Name:</td>
            <td><input name="lastName" type="text" value=""/></td>
            <%-- Associated errors to lastName field displayed --%>
            <td><span name="lastName.errors">Field is required.</span></td>
        </tr>
        <tr>
            <td colspan="3">
                <input type="submit" value="Save Changes"/>
            </td>
        </tr>
    </table>
</form>
```

What if we want to display the entire list of errors for a given page? The example below shows that the`errors`tag also supports some basic wildcarding functionality.

* `path="*"`- displays all errors

* `path="lastName"`- displays all errors associated with the`lastName`field

* if`path`is omitted - object errors only are displayed

The example below will display a list of errors at the top of the page, followed by field-specific errors next to the fields:

```js
<form:form>
    <form:errors path="*" cssClass="errorBox"/>
    <table>
        <tr>
            <td>First Name:</td>
            <td><form:input path="firstName"/></td>
            <td><form:errors path="firstName"/></td>
        </tr>
        <tr>
            <td>Last Name:</td>
            <td><form:input path="lastName"/></td>
            <td><form:errors path="lastName"/></td>
        </tr>
        <tr>
            <td colspan="3">
                <input type="submit" value="Save Changes"/>
            </td>
        </tr>
    </table>
</form:form>
```

The HTML would look like:

```js
<form method="POST">
    <span name="*.errors" class="errorBox">Field is required.<br/>Field is required.</span>
    <table>
        <tr>
            <td>First Name:</td>
            <td><input name="firstName" type="text" value=""/></td>
            <td><span name="firstName.errors">Field is required.</span></td>
        </tr>

        <tr>
            <td>Last Name:</td>
            <td><input name="lastName" type="text" value=""/></td>
            <td><span name="lastName.errors">Field is required.</span></td>
        </tr>
        <tr>
            <td colspan="3">
                <input type="submit" value="Save Changes"/>
            </td>
        </tr>
       </table>
</form>
```

#### HTTP Method Conversion

A key principle of REST is the use of the Uniform Interface. This means that all resources \(URLs\) can be manipulated using the same four HTTP methods: GET, PUT, POST, and DELETE. For each method, the HTTP specification defines the exact semantics. For instance, a GET should always be a safe operation, meaning that is has no side effects, and a PUT or DELETE should be idempotent, meaning that you can repeat these operations over and over again, but the end result should be the same. While HTTP defines these four methods, HTML only supports two: GET and POST. Fortunately, there are two possible workarounds: you can either use JavaScript to do your PUT or DELETE, or simply do a POST with the 'real' method as an additional parameter \(modeled as a hidden input field in an HTML form\). This latter trick is what Spring’s`HiddenHttpMethodFilter`does. This filter is a plain Servlet Filter and therefore it can be used in combination with any web framework \(not just Spring MVC\). Simply add this filter to your web.xml, and a POST with a hidden \_method parameter will be converted into the corresponding HTTP method request.

To support HTTP method conversion the Spring MVC form tag was updated to support setting the HTTP method. For example, the following snippet taken from the updated Petclinic sample

```js
<form:form method="delete">
    <p class="submit"><input type="submit" value="Delete Pet"/></p>
</form:form>
```

This will actually perform an HTTP POST, with the 'real' DELETE method hidden behind a request parameter, to be picked up by the`HiddenHttpMethodFilter`, as defined in web.xml:

```java
<filter>
    <filter-name>httpMethodFilter</filter-name>
    <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
</filter>

<filter-mapping>
    <filter-name>httpMethodFilter</filter-name>
    <servlet-name>petclinic</servlet-name>
</filter-mapping>
```

The corresponding`@Controller`method is shown below:

```java
@RequestMapping(method = RequestMethod.DELETE)
public String deletePet(@PathVariable int ownerId, @PathVariable int petId) {
    this.clinic.deletePet(petId);
    return "redirect:/owners/" + ownerId;
}
```

#### HTML5 Tags

Starting with Spring 3, the Spring form tag library allows entering dynamic attributes, which means you can enter any HTML5 specific attributes.

In Spring 3.1, the form input tag supports entering a type attribute other than 'text'. This is intended to allow rendering new HTML5 specific input types such as 'email', 'date', 'range', and others. Note that entering type='text' is not required since 'text' is the default type.

