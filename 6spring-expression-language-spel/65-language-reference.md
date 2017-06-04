## 6.5 语言参考

### 6.5.1 字面常量表达式

支持的字面常量表达式的类型是字符串，数值（int，real，hex），boolean和null。 字符串由单引号分隔。 要将一个单引号本身放在字符串中，请使用两个单引号。

以下列表显示了字面常量的简单用法。 通常，它们不会像这样使用，而是作为更复杂表达式的一部分，例如在逻辑比较运算符的一侧使用字面常量。

```java
ExpressionParser parser = new SpelExpressionParser();

// evals to "Hello World"
String helloWorld = (String) parser.parseExpression("'Hello World'").getValue();

double avogadrosNumber = (Double) parser.parseExpression("6.0221415E+23").getValue();

// evals to 2147483647
int maxValue = (Integer) parser.parseExpression("0x7FFFFFFF").getValue();

boolean trueValue = (Boolean) parser.parseExpression("true").getValue();

Object nullValue = parser.parseExpression("null").getValue();
```

数字支持使用负号，指数符号和小数点。 默认情况下，使用Double.parseDouble（）解析实数。

### 6.5.2 Properties, Arrays, Lists, Maps, Indexers

使用属性引用进行导航很简单：只需使用句点来指示嵌套的属性值。 `Inventor`类，`pupin`和`tesla`的实例使用示例中使用的类中列出的数据进行填充。 为了导航“down”，并获得Tesla的出生年份和Pupin的出生城市，使用以下表达式。

```java
// evals to 1856
int year = (Integer) parser.parseExpression("Birthdate.Year + 1900").getValue(context);

String city = (String) parser.parseExpression("placeOfBirth.City").getValue(context);
```

属性名称的第一个字母不区分大小写。 数组和列表的内容使用方括号表示法获得。

```java
ExpressionParser parser = new SpelExpressionParser();

// Inventions Array
StandardEvaluationContext teslaContext = new StandardEvaluationContext(tesla);

// evaluates to "Induction motor"
String invention = parser.parseExpression("inventions[3]").getValue(
        teslaContext, String.class);

// Members List
StandardEvaluationContext societyContext = new StandardEvaluationContext(ieee);

// evaluates to "Nikola Tesla"
String name = parser.parseExpression("Members[0].Name").getValue(
        societyContext, String.class);

// List and Array navigation
// evaluates to "Wireless communication"
String invention = parser.parseExpression("Members[0].Inventions[6]").getValue(
        societyContext, String.class);
```

map映射的内容是通过在括号内指定字面常量键值得到的。 在这种情况下，因为Officers映射的键是字符串，我们可以指定字符串字面常量。

```java
// Officer's Dictionary

Inventor pupin = parser.parseExpression("Officers['president']").getValue(
        societyContext, Inventor.class);

// evaluates to "Idvor"
String city = parser.parseExpression("Officers['president'].PlaceOfBirth.City").getValue(
        societyContext, String.class);

// setting values
parser.parseExpression("Officers['advisors'][0].PlaceOfBirth.Country").setValue(
        societyContext, "Croatia");
```

### 6.5.3 内联列表

List列表可以使用`{}`表示法直接在表达式中表示。

```java
// evaluates to a Java list containing the four numbers
List numbers = (List) parser.parseExpression("{1,2,3,4}").getValue(context);

List listOfLists = (List) parser.parseExpression("{{'a','b'},{'x','y'}}").getValue(context);
```

`{}`本身就是一个空列表。 出于性能原因，如果列表本身完全由固定字面常量组成，则会创建一个常量列表来表示表达式，而不是在每个运算操作上构建一个新列表。

### 6.5.4 内联映射

也可以使用`{key：value}`表示法直接在表达式中表示Map映射。

```java
// evaluates to a Java map containing the two entries
Map inventorInfo = (Map) parser.parseExpression("{name:'Nikola',dob:'10-July-1856'}").getValue(context);

Map mapOfMaps = (Map) parser.parseExpression("{name:{first:'Nikola',last:'Tesla'},dob:{day:10,month:'July',year:1856}}").getValue(context);
```

`{：}`本身就是一个空的Map映射。 出于性能原因，如果Map映射本身由固定字面常量或其他嵌套常量结构（List列表或Map映射）组成，则会创建一个常量Map映射来表示表达式，而不是在每个运算操作上构建一个新的Map映射。 引用Map映射键是可选的，上面的示例不使用引用的键。

### 6.5.5 阵列构造

可以使用熟悉的Java语法构建数组，可选择提供一个初始化器，以便在构建时填充数组。

```java
int[] numbers1 = (int[]) parser.parseExpression("new int[4]").getValue(context);

// Array with initializer
int[] numbers2 = (int[]) parser.parseExpression("new int[]{1,2,3}").getValue(context);

// Multi dimensional array
int[][] numbers3 = (int[][]) parser.parseExpression("new int[4][5]").getValue(context);
```

当构造多维数组时，它当前不允许提供初始化器。

### 6.5.6 方法

使用典型的Java编程语法调用方法。 您也可以调用字面常量的方法。

```java
// string literal, evaluates to "bc"
String c = parser.parseExpression("'abc'.substring(2, 3)").getValue(String.class);

// evaluates to true
boolean isMember = parser.parseExpression("isMember('Mihajlo Pupin')").getValue(
        societyContext, Boolean.class);
```

### 6.5.7 运算符

#### 关系运算符

关系运算符 小于，小于等于，大于，大于等于，等于，不等于，使用标准运算符符号表示。

```java
// evaluates to true
boolean trueValue = parser.parseExpression("2 == 2").getValue(Boolean.class);

// evaluates to false
boolean falseValue = parser.parseExpression("2 < -5.0").getValue(Boolean.class);

// evaluates to true
boolean trueValue = parser.parseExpression("'black' < 'block'").getValue(Boolean.class);
```

| ![\[Note\]](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| --- |
| 大于/小于与null的比较遵循一个简单的规则：null在这里被视为没有（不是0）。 因此，任何其他值始终大于null（X&gt; null始终为真），并且没有其他值比它小（X &lt;null始终为false）。如果您更喜欢数字比较，请避免基于数字的null比较，这有利于与零比较（例如X&gt; 0或X &lt;0）。 |

除标准关系运算符外，SpEL还支持`instanceof` 和基于正则表达式的匹配运算符。

```java
// evaluates to false
boolean falseValue = parser.parseExpression(
        "'xyz' instanceof T(Integer)").getValue(Boolean.class);

// evaluates to true
boolean trueValue = parser.parseExpression(
        "'5.00' matches '\^-?\\d+(\\.\\d{2})?$'").getValue(Boolean.class);

//evaluates to false
boolean falseValue = parser.parseExpression(
        "'5.0067' matches '\^-?\\d+(\\.\\d{2})?$'").getValue(Boolean.class);
```

| ![\[Note\]](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| --- |
| 请谨慎使用原始类型，因为它们会立即包装到包装器类型中，因此，如果预期的话，`1 instanceof T(int)`将计算为false，而`1 instanceof T(Integer)`的计算结果为true。 |

每个符号操作符也可以被指定为纯粹的字母等价物。 这避免了所使用的符号对嵌入表达式的文档类型（例如XML文档）具有特殊含义的问题。 文本等价物如下所示：lt（&lt;），gt（&gt;），le（⇐），ge（&gt; =），eq（==），ne（！=），div（/），mod（％） not（！）。 这些不区分大小写。

#### 逻辑运算符

支持的逻辑运算符是and, or, and not。 它们的用途如下所示。

```java
// -- AND --

// evaluates to false
boolean falseValue = parser.parseExpression("true and false").getValue(Boolean.class);

// evaluates to true
String expression = "isMember('Nikola Tesla') and isMember('Mihajlo Pupin')";
boolean trueValue = parser.parseExpression(expression).getValue(societyContext, Boolean.class);

// -- OR --

// evaluates to true
boolean trueValue = parser.parseExpression("true or false").getValue(Boolean.class);

// evaluates to true
String expression = "isMember('Nikola Tesla') or isMember('Albert Einstein')";
boolean trueValue = parser.parseExpression(expression).getValue(societyContext, Boolean.class);

// -- NOT --

// evaluates to false
boolean falseValue = parser.parseExpression("!true").getValue(Boolean.class);

// -- AND and NOT --
String expression = "isMember('Nikola Tesla') and !isMember('Mihajlo Pupin')";
boolean falseValue = parser.parseExpression(expression).getValue(societyContext, Boolean.class);
```

#### 算术运算符

加法运算符可以用于数字和字符串。 减法，乘法和除法只能用于数字。 支持的其他算术运算符是模数（％）和指数幂（^）。 执行标准运算符优先级。 这些操作符将在下面展示。

```java
// Addition
int two = parser.parseExpression("1 + 1").getValue(Integer.class); // 2

String testString = parser.parseExpression(
        "'test' + ' ' + 'string'").getValue(String.class); // 'test string'

// Subtraction
int four = parser.parseExpression("1 - -3").getValue(Integer.class); // 4

double d = parser.parseExpression("1000.00 - 1e4").getValue(Double.class); // -9000

// Multiplication
int six = parser.parseExpression("-2 * -3").getValue(Integer.class); // 6

double twentyFour = parser.parseExpression("2.0 * 3e0 * 4").getValue(Double.class); // 24.0

// Division
int minusTwo = parser.parseExpression("6 / -3").getValue(Integer.class); // -2

double one = parser.parseExpression("8.0 / 4e0 / 2").getValue(Double.class); // 1.0

// Modulus
int three = parser.parseExpression("7 % 4").getValue(Integer.class); // 3

int one = parser.parseExpression("8 / 5 % 2").getValue(Integer.class); // 1

// Operator precedence
int minusTwentyOne = parser.parseExpression("1+2-3*8").getValue(Integer.class); // -21
```

### 6.5.8 赋值

通过使用赋值运算符来完成属性的设置。 这通常在调用`setValue`之前完成，但也可以在对`getValue`的调用中完成。

```java
Inventor inventor = new Inventor();
StandardEvaluationContext inventorContext = new StandardEvaluationContext(inventor);

parser.parseExpression("Name").setValue(inventorContext, "Alexander Seovic2");

// alternatively

String aleks = parser.parseExpression(
        "Name = 'Alexandar Seovic'").getValue(inventorContext, String.class);
```

### 6.5.9 类型运算符

特殊的T运算符可用于指定`java.lang.Class`（类型）的实例。 也可以使用此运算符调用静态方法。`TheStandardEvaluationContext`使用`TypeLocator`来查找类型，并且可以使用对`java.lang`包的理解来构建`StandardTypeLocator`（可以被替换）。 这意味着对`java.lang`中的类型的引用`T（）`不需要是完全限定的，但是所有其他类型引用必须是。

```java
Class dateClass = parser.parseExpression("T(java.util.Date)").getValue(Class.class);

Class stringClass = parser.parseExpression("T(String)").getValue(Class.class);

boolean trueValue = parser.parseExpression(
        "T(java.math.RoundingMode).CEILING < T(java.math.RoundingMode).FLOOR")
        .getValue(Boolean.class);
```

### 6.5.10 构造函数

可以使用新的运算符调用构造函数。 除了原始类型和字符串（可以使用int，float等）之外，所有标准类名称都应该被使用。

```java
Inventor einstein = p.parseExpression(
        "new org.spring.samples.spel.inventor.Inventor('Albert Einstein', 'German')")
        .getValue(Inventor.class);

//create new inventor instance within add method of List
p.parseExpression(
        "Members.add(new org.spring.samples.spel.inventor.Inventor(
            'Albert Einstein', 'German'))").getValue(societyContext);
```

### 6.5.11 变量

可以使用语法`#variableName`在表达式中引用变量。 使用`StandardEvaluationContext`上的`setVariable`方法设置变量。

```java
Inventor tesla = new Inventor("Nikola Tesla", "Serbian");
StandardEvaluationContext context = new StandardEvaluationContext(tesla);
context.setVariable("newName", "Mike Tesla");

parser.parseExpression("Name = #newName").getValue(context);

System.out.println(tesla.getName()) // "Mike Tesla"
```

#### \#this 和 \#root 变量

变量\#this始终被定义，并且引用当前的运算操作对象（针对被解析的那个非限定引用）。 变量\#root始终被定义，并引用根上下文对象。 虽然\#this可能因为表达式的组件被运算操作而变化，但是\#root总是引用根。

```java
// create an array of integers
List<Integer> primes = new ArrayList<Integer>();
primes.addAll(Arrays.asList(2,3,5,7,11,13,17));

// create parser and set variable 'primes' as the array of integers
ExpressionParser parser = new SpelExpressionParser();
StandardEvaluationContext context = new StandardEvaluationContext();
context.setVariable("primes",primes);

// all prime numbers > 10 from the list (using selection ?{...})
// evaluates to [11, 13, 17]
List<Integer> primesGreaterThanTen = (List<Integer>) parser.parseExpression(
        "#primes.?[#this>10]").getValue(context);
```

### 6.5.12 功能

您可以通过注册能够在表达式字符串中调用的用户定义的函数来扩展SpEL。 该功能通过方法使用`StandardEvaluationContext`进行注册。

```java
public void registerFunction(String name, Method m)
```

对Java方法的引用提供了该函数的实现。 例如，一个反转字符串的实用方法如下所示。

```java
public abstract class StringUtils {

    public static String reverseString(String input) {
        StringBuilder backwards = new StringBuilder();
        for (int i = 0; i < input.length(); i++)
            backwards.append(input.charAt(input.length() - 1 - i));
        }
        return backwards.toString();
    }
}
```

然后将该方法注册到运算操作的上下文中，并可在表达式字符串中使用。

```java
ExpressionParser parser = new SpelExpressionParser();
StandardEvaluationContext context = new StandardEvaluationContext();

context.registerFunction("reverseString",
    StringUtils.class.getDeclaredMethod("reverseString", new Class[] { String.class }));

String helloWorldReversed = parser.parseExpression(
    "#reverseString('hello')").getValue(context, String.class);
```

### 6.5.13 Bean引用

如果使用bean解析器配置了运算操作上下文，则可以使用（@）符号从表达式中查找bean。

```java
ExpressionParser parser = new SpelExpressionParser();
StandardEvaluationContext context = new StandardEvaluationContext();
context.setBeanResolver(new MyBeanResolver());

// This will end up calling resolve(context,"foo") on MyBeanResolver during evaluation
Object bean = parser.parseExpression("@foo").getValue(context);
```

要访问工厂bean本身，bean名称应改为带有（＆）符号的前缀。

```java
ExpressionParser parser = new SpelExpressionParser();
StandardEvaluationContext context = new StandardEvaluationContext();
context.setBeanResolver(new MyBeanResolver());

// This will end up calling resolve(context,"&foo") on MyBeanResolver during evaluation
Object bean = parser.parseExpression("&foo").getValue(context);
```

### 6.5.14 三元运算符 \(If-Then-Else\)

您可以使用三元运算符在表达式中执行if-then-else条件逻辑。 一个最小的例子是：

```java
String falseString = parser.parseExpression(
        "false ? 'trueExp' : 'falseExp'").getValue(String.class);
```

在这种情况下，布尔值false会返回字符串值“falseExp”。 一个更现实的例子如下所示。

```java
parser.parseExpression("Name").setValue(societyContext, "IEEE");
societyContext.setVariable("queryName", "Nikola Tesla");

expression = "isMember(#queryName)? #queryName + ' is a member of the ' " +
        "+ Name + ' Society' : #queryName + ' is not a member of the ' + Name + ' Society'";

String queryResultString = parser.parseExpression(expression)
        .getValue(societyContext, String.class);
// queryResultString = "Nikola Tesla is a member of the IEEE Society"
```

另请参阅Elvis操作员的下一部分，为三元运算符提供更短的语法。

### 6.5.15 Elvis操作符

Elvis操作符缩短了三元操作符语法，并以Groovy语言使用。 通过三元运算符语法，您通常必须重复一次变量两次，例如：

```java
String name = "Elvis Presley";
String displayName = name != null ? name : "Unknown";
```

相反，您可以使用Elvis操作符，命名与Elvis的发型相似。

```java
ExpressionParser parser = new SpelExpressionParser();

String name = parser.parseExpression("name?:'Unknown'").getValue(String.class);

System.out.println(name); // 'Unknown'
```

这里是一个更复杂的例子。

```java
ExpressionParser parser = new SpelExpressionParser();

Inventor tesla = new Inventor("Nikola Tesla", "Serbian");
StandardEvaluationContext context = new StandardEvaluationContext(tesla);

String name = parser.parseExpression("Name?:'Elvis Presley'").getValue(context, String.class);

System.out.println(name); // Nikola Tesla

tesla.setName(null);

name = parser.parseExpression("Name?:'Elvis Presley'").getValue(context, String.class);

System.out.println(name); // Elvis Presley
```

### 6.5.16 安全导航运算符

安全导航运算符用于避免`NullPointerException`并来自Groovy语言。 通常当您对对象的引用时，您可能需要在访问对象的方法或属性之前验证它不为空。 为了避免这种情况，安全导航运算符将简单地返回null而不是抛出异常。

```java
ExpressionParser parser = new SpelExpressionParser();

Inventor tesla = new Inventor("Nikola Tesla", "Serbian");
tesla.setPlaceOfBirth(new PlaceOfBirth("Smiljan"));

StandardEvaluationContext context = new StandardEvaluationContext(tesla);

String city = parser.parseExpression("PlaceOfBirth?.City").getValue(context, String.class);
System.out.println(city); // Smiljan

tesla.setPlaceOfBirth(null);

city = parser.parseExpression("PlaceOfBirth?.City").getValue(context, String.class);

System.out.println(city); // null - does not throw NullPointerException!!!
```

| ![\[Note\]](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| --- |
| Elvis操作符可用于在表达式中设置默认值，例如 在@Value表达式中：`@Value("#{systemProperties['pop3.port'] ?: 25}")`这将注入系统属性`pop3.port`（如果已定义）或25（如果未定义）。 |

### 6.5.17 集合选择

选择是强大的表达式语言功能，允许您通过从其条目中选择将一些源集合转换为另一个。

选择使用语法`.?[selectionExpression]`。 这将过滤收集并返回一个包含原始元素子集的新集合。 例如，选择将使我们能够轻松得到Serbian发明家名单：

```java
List<Inventor> list = (List<Inventor>) parser.parseExpression(
        "Members.?[Nationality == 'Serbian']").getValue(societyContext);
```

list列表和map映射都可以进行选择。 在前一种情况下，根据每个单独列表元素，同时针对map映射，根据每个map映射条目（Java类型`Map.Entry`的对象）运算操作作为选择标准。 map映射条目的键和值可以作为选择中使用的属性访问。

此表达式将返回一个由原始map映射的元素组成的新map映射，其中条目值小于27。

```java
Map newMap = parser.parseExpression("map.?[value<27]").getValue();
```

除了返回所有选定的元素之外，还可以检索第一个或最后一个值。 要获得与选择匹配的第一个条目，语法为`^[…]`，同时获取最后匹配的选择，语法为`$[…]`。

### 6.5.18 集合投影

投影允许集合驱动子表达式的运算操作，结果是一个新的集合。 投影的语法是`![projectionExpression]`。 最容易理解的例子，假设我们有一个发明家的名单，但希望得到他们出生的城市的名单。 有效地，我们要对发明人列表中的每个条目进行“placeOfBirth.city”运算操作。 使用投影：

```java
// returns ['Smiljan', 'Idvor' ]
List placesOfBirth = (List)parser.parseExpression("Members.![placeOfBirth.city]");
```

map映射也可以用于驱动投影，在这种情况下，投影表达式将针对map映射中的每个条目进行运算操作（表示为Java `Map.Entry`）。 跨map映射投影的结果是由对每个map映射条目的投影表达式的运算操作组成的列表。

### 6.5.19 表达式模板

表达式模板允许文字文本与一个或多个运算操作块进行混合。 每个运算操作块都用您可以定义的前缀和后缀字符进行分隔，常用的选择是使用`#{ }`作为分隔符。 例如，

```java
String randomPhrase = parser.parseExpression(
        "random number is #{T(java.lang.Math).random()}",
        new TemplateParserContext()).getValue(String.class);

// evaluates to "random number is 0.7038186818312008"
```

字符串通过连接文本文本'random number is'与运算操作＃{}分隔符中的表达式的结果进行运算操作，在这种情况下是调用random（）方法的结果。 `parseExpression（）`方法的第二个参数是`ParserContext`类型。 `ParserContext`接口用于决定表达式如何被解析以支持表达式模板功能。 `TemplateParserContext`的定义如下所示。

```java
public class TemplateParserContext implements ParserContext {

    public String getExpressionPrefix() {
        return "#{";
    }

    public String getExpressionSuffix() {
        return "}";
    }

    public boolean isTemplate() {
        return true;
    }
}
```



