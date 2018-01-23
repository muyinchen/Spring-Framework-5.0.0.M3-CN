### 19.4.5 绑定支持和表单处理

Spring提供了一个用于JSP的标签库，其中包含（其中包含）`<spring:bind/>`标签。 此标记主要允许表单从表单支持对象显示值，并显示来自Web或业务层的`Validator`的失败验证结果。 Spring还支持FreeMarker中的相同功能，还有其他方便的宏用于生成表单输入元素。

#### 绑定macros

在这两种语言的`spring-webmvc.jar`文件中都保留了一组标准的macros，因此它们始终可供适当配置的应用程序使用。

在Spring库中定义的一些macros被认为是内部的（私有的），但macros定义中不存在这样的范围，使调用代码和用户模板的所有macros都可见。 以下部分仅关注您需要从模板中直接调用的macros。 如果您希望直接查看macros代码，则在包`org.springframework.web.servlet.view.freemarker`中将该文件称为`spring.ftl`。

#### 简单绑定

在作为Spring MVC控制器表单视图的HTML表单（vm / ftl模板）中，可以使用与以下类似的代码绑定到字段值，并以与JSP等效的方式类似的方式显示每个输入字段的错误消息。 下面显示了以前配置的`personForm`视图的示例代码：

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

`<@spring.bind>`需要一个'path'参数，它由你的命令对象的名字组成（除非你在FormController属性中改变它，否则它将是'command'），后面跟着一个句点和命令中的字段名 你想绑定的对象。 也可以使用嵌套字段，如“command.address.street”。 `bind`macro采用由`web.xml`中的ServletContext参数`defaultHtmlEscape`指定的默认HTML转义行为。

macro的可选形式称为`<@spring.bindEscaped>`接受第二个参数，并明确指定是否应该在状态错误消息或值中使用HTML转义。 根据需要设置为true或false。 额外的表单处理macro简化了HTML转义的使用，应尽可能使用这些macro。 他们在下一节解释。

#### 表单输入生成 macros

这两种语言的其他便利macros简化了绑定和表单生成（包括验证错误显示）。 从来没有必要使用这些macros来生成表单输入字段，并且可以将它们与简单的HTML混合并进行匹配，或者直接调用之前突出显示的弹簧绑定macros。

下面的可用macro表显示了每个VTL和FTL定义以及每个参数列表。

**Table 19.1. Table of macro definitions**

| macro | FTL 定义 | **消息** | \(根据代码参数从资源束输出字符串\) |
| :--- | :--- | :--- | :--- |
|  | &lt;@spring.message code/&gt; | **messageText**\(根据代码参数从资源束输出一个字符串，退回到默认参数的值\) | &lt;@spring.messageText code, text/&gt; |
|  | **url**\(使用应用程序的上下文根前缀相对URL\) | &lt;@spring.url relativeUrl/&gt; | **formInput**\(用于收集用户输入的标准输入字段\) |
|  | &lt;@spring.formInput path, attributes, fieldType/&gt; | **formHiddenInput**\* \(用于提交非用户输入的隐藏输入字段\) | &lt;@spring.formHiddenInput path, attributes/&gt; |
|  | **formPasswordInput**\* \(用于收集密码的标准输入字段。 请注意，不会在此类型的字段中填充任何值\) | &lt;@spring.formPasswordInput path, attributes/&gt; | **formTextarea**\(大文本字段用于收集长，自由格式的文本输入\) |
|  | &lt;@spring.formTextarea path, attributes/&gt; | **formSingleSelect**\(下拉框选项允许选择单个所需的值\) | &lt;@spring.formSingleSelect path, options, attributes/&gt; |
|  | **formMultiSelect**\(允许用户选择0个或更多值的选项列表框\) | &lt;@spring.formMultiSelect path, options, attributes/&gt; | **formRadioButtons**\(一组单选按钮允许从可用选项中进行单次选择\) |
|  | &lt;@spring.formRadioButtons path, options separator, attributes/&gt; | **formCheckboxes**\(一组允许选择0个或更多值的复选框\) | &lt;@spring.formCheckboxes path, options, separator, attributes/&gt; |
|  | **formCheckbox**\(a single checkbox\) | &lt;@spring.formCheckbox path, attributes/&gt; | **showErrors**\(简化显示绑定字段的验证错误\) |

* 在FTL（FreeMarker）中，这两个macro实际上并不是必需的，因为您可以使用常规`formInput`宏指定“hidden”或“password”作为`fieldType`参数的值。

上述任何一个macro的参数都有一致的含义：

* path: 要绑定的字段的名称（即“command.name”）

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



