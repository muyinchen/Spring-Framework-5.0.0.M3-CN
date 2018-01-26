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

* 选项：可以在输入字段中选择的所有可用值的映射。 映射关键字表示将从表单中返回并绑定到命令对象的值。 映射存储在键上的映射对象是表单上显示给用户的标签，可能与表单发回的相应值不同。 通常这种映射是由控制器提供的参考数据。 任何Map实现可以根据需求的行为选择。 对于严格排序的映射，可以使用`SortedMap`（如带有合适的Comparator的`TreeMap`），对于任何应该以插入顺序返回值的Maps，请使用`LinkedHashMap`或Commons-Collection中的`LinkedMap`。

* 分隔符：其中多个选项可用作离散元素（单选按钮或复选框），用于分隔列表中的每一个（即“&lt;br&gt;”）的字符序列。

* 属性：包含在HTML标签本身内的任意标签或文本的附加字符串。 这个字符串被macro指令回显。 例如，在textarea字段中您可以将属性设置为“rows =”5“ cols =”60”或者可以传递样式信息，例如'style="border:1px solid silver"'。

* classOrStyle：对于showErrors宏，包含每个错误的span标签将使用的CSS类的名称。如果没有提供信息（或值为空），那么错误将被包裹在&lt;b&gt;&lt;/ b&gt;标签中。

这些宏的例子在FTL中列出了一些，在VTL中列出了一些。 在两种语言之间存在使用差异的地方，他们在笔记中解释。

##### 输入的值

formInput宏接受路径参数（command.name）和一个在上例中为空的附加属性参数。 该宏以及所有其他表单生成宏，对路径参数执行隐式弹簧绑定。 绑定保持有效，直到发生新的绑定为止，所以showErrors宏不需要再次传递路径参数 - 它只是在绑定上次创建的任何字段上运行。

showErrors宏需要一个分隔符参数（将用于分隔给定字段上的多个错误的字符），并且还接受第二个参数，这次是类名称或样式属性。 请注意，FreeMarker能够为attributes参数指定默认值。

```java
<@spring.formInput "command.name"/>
<@spring.showErrors "<br>"/>
```

输出显示在生成名称字段的表单片段中，并且在字段中提交表单时没有值时显示验证错误。 验证通过Spring的验证框架进行。

生成的HTML如下所示：

```java
Name:
<input type="text" name="name" value="">
<br>
    <b>required</b>
<br>
<br>
```

formTextarea宏与formInput宏的工作方式相同，并接受相同的参数列表。 通常，第二个参数（属性）将用于传递文本区域的样式信息或行和列属性。

##### 选择字段

四个选择字段宏可用于在HTML表单中生成常见的UI值选择输入。

* formSingleSelect

* formMultiSelect

* formRadioButtons

* formCheckboxes

四个宏中的每一个都接受一个包含表单字段值的选项的Map，以及与该值相对应的标签。 值和标签可以是相同的。

FTL中的单选按钮示例如下。 表单支持对象为该字段指定默认值'London'，因此不需要验证。 当表单呈现时，整个城市列表将作为模型中的参考数据以'cityMap'名称提供。

```java
...
Town:
<@spring.formRadioButtons "command.address.town", cityMap, ""/><br><br>
```

这将呈现一行单选按钮，每个城市地图中的值使用分隔符""。 没有提供额外的属性（缺少宏的最后一个参数）。 cityMap对地图中的每个键值对使用相同的字符串。 地图的键是表单实际提交的POST请求参数，map值是用户看到的标签。 在上面的例子中，给定一个三个知名城市的列表，并在窗体支持对象中使用一个默认值，HTML将会是

```js
Town:
<input type="radio" name="address.town" value="London">London</input>
<input type="radio" name="address.town" value="Paris" checked="checked">Paris</input>
<input type="radio" name="address.town" value="New York">New York</input>
```

例如，如果您的应用程序希望通过内部代码处理城市，则可以使用适当的键（如下面的示例）创建代码map。

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

现在代码将产生输出，其中无线电值是相关的代码，但是用户仍然看到更多用户友好的城市名称。

```js
Town:
<input type="radio" name="address.town" value="LDN">London</input>
<input type="radio" name="address.town" value="PRS" checked="checked">Paris</input>
<input type="radio" name="address.town" value="NYC">New York</input>
```

#### HTML转义和符合XHTML标准

上面的表单宏的默认使用将导致符合HTML 4.01的HTML标记，并使用Spring绑定支持所使用的web.xml中定义的HTML转义的默认值。 为了使标记符合XHTML或覆盖默认的HTML转义值，您可以在模板中指定两个变量（或者在您的模型中，您的模板中可以看到它们）。 在模板中指定它们的好处是可以在模板处理中稍后将其更改为不同的值，以便为表单中的不同字段提供不同的行为。

要切换到标记的XHTML合规性，请为名为xhtmlCompliant的模型/上下文变量指定值“true”：

```java
<#-- for FreeMarker -->
<#assign xhtmlCompliant = true in spring>
```

在处理此指令后，由Spring宏生成的任何标签现在都将符合XHTML标准。

以类似的方式，每个字段可以指定HTML转义：

```java
<#-- until this point, default HTML escaping is used -->

<#assign htmlEscape = true in spring>
<#-- next field will use HTML escaping -->
<@spring.formInput "command.name"/>

<#assign htmlEscape = false in spring>
<#-- all future fields will be bound with HTML escaping off -->
```



