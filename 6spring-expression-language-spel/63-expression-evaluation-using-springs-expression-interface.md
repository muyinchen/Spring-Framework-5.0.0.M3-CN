## 6.3 使用Spring表达式接口的表达式运算操作

本节介绍了SpEL接口及其表达式语言的简单使用。完整的语言参考可以在“语言参考”一节中找到。

以下代码介绍了SpEL API来运算操作文字字符串表达式“Hello World”。

```java
ExpressionParser parser = new SpelExpressionParser();
Expression exp = parser.parseExpression("'Hello World'");
String message = (String) exp.getValue();
```

变量message的值只是 'Hello World'.

您最有可能使用的SpEL类和接口位于包 `org.springframework.expression` 及其子包和包`spel.support`

接口`ExpressionParser`负责解析表达式字符串。 在此示例中，表达式字符串是由周围的单引号表示的字符串文字。接口`Expression`负责运算操作先前定义的表达式字符串。 当分别调用`parser.parseExpression`和`exp.getValue`时，有两个可以抛出的异常，`ParseException`和`EvaluationException`。

SpEL支持各种功能，如调用方法，访问属性和调用构造函数。

作为方法调用的一个例子，我们在字符串文字中调用`concat`方法。

```java
ExpressionParser parser = new SpelExpressionParser();
Expression exp = parser.parseExpression("'Hello World'.concat('!')");
String message = (String) exp.getValue();
```

变量message的值现在是 'Hello World!'.

作为调用JavaBean属性的示例，可以调用String属性`Bytes`，如下所示。

```java
ExpressionParser parser = new SpelExpressionParser();

// invokes 'getBytes()'
Expression exp = parser.parseExpression("'Hello World'.bytes");
byte[] bytes = (byte[]) exp.getValue();
```

Spel还支持嵌套属性，使用标准点符号，即prop1.prop2.prop3链式写法和属性值的设置

Public fields may also be accessed.

```java
ExpressionParser parser = new SpelExpressionParser();

// invokes 'getBytes().length'
Expression exp = parser.parseExpression("'Hello World'.bytes.length");
int length = (Integer) exp.getValue();
```

可以调用String的构造函数，而不是使用字符串文字。

```java
ExpressionParser parser = new SpelExpressionParser();
Expression exp = parser.parseExpression("new String('hello world').toUpperCase()");
String message = exp.getValue(String.class);
```

注意使用通用方法`public（T）T getValue（Class <T> desiredResultType）`。 使用此方法不需要将表达式的值转换为所需的结果类型。 如果该值不能转换为类型`T`或使用注册的类型转换器转换，则将抛出`EvaluationException`。

Spel的更常见的用法是提供一个针对特定对象实例（称为根对象）进行运算操作的表达式字符串。 这里有两个选项 ，并且选择哪个由反对当前被验证的表示式的对象是否在每次调用后而改变再验证表达式来决定。在以下示例中，我们从`Inventor`类的实例中检索`name`属性。

```java
// 创建并对日历对象设值
GregorianCalendar c = new GregorianCalendar();
c.set(1856, 7, 9);

// 构造函数的参数是姓名，生日和国籍。
Inventor tesla = new Inventor("Nikola Tesla", c.getTime(), "Serbian");

ExpressionParser parser = new SpelExpressionParser();
Expression exp = parser.parseExpression("name");

EvaluationContext context = new StandardEvaluationContext(tesla);
String name = (String) exp.getValue(context);
```

在最后一行，字符串变量name的值将被设置为“Nikola Tesla”。 `StandardEvaluationContext`类可以指定哪个对象的“name”属性将被运算操作。 如果根对象不太可能改变，这是使用的机制，可以在运算操作上下文中简单地设置一次。 如果根对象可能会重复更改，则可以在每次调用getValue时提供该对象，如下例所示：

```java
// 创建并对日历对象设值
GregorianCalendar c = new GregorianCalendar();
c.set(1856, 7, 9);

// 构造函数的参数是姓名，生日和国籍。
Inventor tesla = new Inventor("Nikola Tesla", c.getTime(), "Serbian");

ExpressionParser parser = new SpelExpressionParser();
Expression exp = parser.parseExpression("name");
String name = (String) exp.getValue(tesla);
```

在这种情况下，目标对象`tesla`已经直接提供给`getValue`，表达式运算操作的基础动作即在内部创建和管理默认运算操作的上下文 - 它不需要被额外提供。

`StandardEvaluationContext`构建起来代价相对昂贵，并在重复使用时构建高速缓存的状态，使得能够更快地执行后续的表达式运算操作。 因此，最好在可能的情况下缓存和重新使用它们，而不是为每个表达式运算操作构建一个新的。

在某些情况下，可能需要使用配置的运算操作上下文，但在每次调用`getValue`时仍会提供不同的根对象。 `getValue`允许在同一个调用中指定两者。 在这些情况下，传递给该调用的根对象被认为覆盖在任何（可能为null）指定的运算操作的上下文中。

| ![\[Note\]](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| --- |
| 在独立使用SpEL中，需要创建解析器，解析表达式，并可能提供运算操作上下文和根上下文对象。 但是，更常见的用法是仅将SpEL表达式字符串作为配置文件的一部分，例如Spring bean或Spring Web Flow定义。 在这种情况下，解析器，运算操作上下文，根对象和任何预定义的变量都是隐式设置的，要求用户除表达式之外不要指定。 |

作为最后的介绍性示例，使用上一个示例中的Inventor对象来显示布尔运算符的使用。

```java
Expression exp = parser.parseExpression("name == 'Nikola Tesla'");
boolean result = exp.getValue(context, Boolean.class); // evaluates to true
```

### 6.3.1 EvaluationContext 接口

运算表达式以解析属性，方法，字段并帮助执行类型转换时使用`EvaluationContext`接口。 开箱即用的实现`StandardEvaluationContext`，使用反射来操纵对象，并缓存`java.lang.reflect.Method`，`java.lang.reflect.Field`和`java.lang.reflect.Constructor`实例以提高性能。

`StandardEvaluationContext`是您可以通过方法`setRootObject（）`或将根对象传递到构造函数中来指定要对其进行运算操作的根对象。 您还可以使用`setVariable（）`和`registerFunction（）`方法指定将在表达式中使用的变量和函数。 变量和函数的使用在语言参考部分变量和函数中有所描述。 `StandardEvaluationContext`还可以注册自定义`ConstructorResolvers`，`MethodResolvers`和`PropertyAccessors`，以扩展SpEL如何运算操作表达式。 请参考这些类的JavaDoc了解更多详细信息。

#### 类型转换

默认情况下，SpEL使用Spring核心`（org.springframework.core.convert.ConversionService）`中提供的转换服务。 此转换服务附带许多转换器，内置于常用转换，但也可完全扩展，因此可以添加类型之间的自定义转换。 此外，它具有泛型感知的关键功能。 这意味着在使用表达式中的泛型类型时，SpEL将尝试转换以维护遇到的任何对象的类型正确性。

这在实践中意味着什么？ 假设使用`setValue（）`的赋值被用于设置`List`属性。 属性的类型实际上是`List <Boolean>`。 Spel将会认识到列表的元素需要在被放置在其中之前被转换为布尔值。 一个简单的例子：

```java
class Simple {
    public List<Boolean> booleanList = new ArrayList<Boolean>();
}

Simple simple = new Simple();

simple.booleanList.add(true);

StandardEvaluationContext simpleContext = new StandardEvaluationContext(simple);

// false允许作为字符串， SpEL和转换服务将正确识别它需要是一个布尔值并对其进行转换
parser.parseExpression("booleanList[0]").setValue(simpleContext, "false");

// b为布尔值false
Boolean b = simple.booleanList.get(0);
```

### 6.3.2 解析器配置

可以使用解析器配置对象`（org.springframework.expression.spel.SpelParserConfiguration）`来配置SpEL表达式解析器。 该配置对象控制一些表达式组件的行为。 例如，如果索引到数组或集合中，并且指定索引处的元素为空，其将自动创建元素。 当使用由一组属性引用组成的表达式时，这是非常有用的。 如果索引到数组或列表中，并指定超出数组或列表的当前大小的结尾的索引，其将自动提高数组或列表大小以适应该索引。

```java
class Demo {
    public List<String> list;
}

// 开启如下功能:
// - 空引用自动初始化
// - 集合大小自动增长
SpelParserConfiguration config = new SpelParserConfiguration(true,true);

ExpressionParser parser = new SpelExpressionParser(config);

Expression expression = parser.parseExpression("list[3]");

Demo demo = new Demo();

Object o = expression.getValue(demo);

// demo.list现在被实例化为拥有四个元素的集合
// 每个元素都是一个新的空字符串
```

还可以配置SpEL表达式编译器的行为。

### 6.3.3 SpEL编译

Spring Framework 4.1包含一个基本的表达式编译器。 表达式通常因为在运算操作过程中提供了大量的动态灵活性被解释，但不能提供最佳性能。 对于偶尔的表达式使用这是很好的，但是当被其他并不真正需要动态灵活性的组件（如Spring Integration）使用时，性能可能非常重要。

新的SpEL编译器旨在满足这一需求。 编译器将在体现了表达行为的运算操作期间即时生成一个真正的Java类，并使用它来实现更快的表达式求值。 由于缺少对表达式按类型归类，编译器在执行编译时会使用在表达式解释运算期间收集的信息来编译。 例如，它不仅仅是从表达式中知道属性引用的类型，而是在第一个解释运算过程中会发现它是什么。 当然，如果各种表达式元素的类型随着时间的推移而变化，那么基于此信息的编译可能会导致问题产生。 因此，编译最适合于重复运算操作时类型信息不会改变的表达式。

对于这样的基本表达式：

`someArray[0].someProperty.someOtherProperty < 0.1`

这涉及到数组访问，某些属性的取消和数字操作，性能增益可以非常显着。 在50000次迭代的微型基准运行示例中，只使用解释器需要75ms，而使用编译版本的表达式仅需3ms。

#### 编译器配置

默认情况下，编译器未打开，但有两种方法可以打开它。 可以使用先前讨论的解析器配置过程或者当将SpEL使用嵌入到另一个组件中时通过系统属性打开它。 本节讨论这两个选项。

重要的是要明白，编译器可以运行几种模式，在枚举`（org.springframework.expression.spel.SpelCompilerMode）`中捕获。 模式如下：

* `OFF` - 编译器关闭; 这是默认值。
* `IMMEDIATE` - 在即时模式下，表达式将尽快编译。 这通常是在第一次解释运算之后。 如果编译的表达式失败（通常是由于类型更改，如上所述）引起的，则表达式运算操作的调用者将收到异常。
* `MIXED` - 在混合模式下，表达式随着时间的推移在解释模式和编译模式之间静默地切换。 经过一些解释运行后，它们将切换到编译模式，如果编译后的表单出现问题（如上所述改变类型），表达式将自动重新切换回解释模式。 稍后，它可能生成另一个编译表单并切换到它。 基本上，用户进入即时模式的异常是内部处理的。 

存在IMMEDIATE模式，因为混合模式可能会导致具有副作用的表达式的问题。 如果一个编译的表达式在部分成功之后崩掉，它可能已经完成了影响系统状态的事情。 如果发生这种情况，调用者可能不希望它在解释模式下静默地重新运行，因为表达式的一部分可能运行两次。

选择模式后，使用`SpelParserConfiguration`配置解析器：

```java
SpelParserConfiguration config = new SpelParserConfiguration(SpelCompilerMode.IMMEDIATE,
    this.getClass().getClassLoader());

SpelExpressionParser parser = new SpelExpressionParser(config);

Expression expr = parser.parseExpression("payload");

MyMessage message = new MyMessage();

Object payload = expr.getValue(message);
```

当指定编译器模式时，也可以指定一个类加载器（允许传递null）。 编译表达式将在任何提供的子类加载器中被定义。 重要的是确保是否指定了类加载器，它可以看到表达式运算操作过程中涉及的所有类型。 如果没有指定，那么将使用默认的类加载器（通常是在表达式计算期间运行的线程的上下文类加载器）。

配置编译器的第二种方法是将SpEL嵌入其他组件内部使用，并且可能无法通过配置对象进行配置。 在这些情况下，可以使用系统属性。 属性`spring.expression.compiler.mode`可以设置为`SpelCompilerMode`枚举值之一（关闭，即时或混合）。

#### 编译器限制

虽然Spring Framework 4.1的基本编译框架已经存在， 但是，框架还不支持编译各种表达式。 最初的重点是在可能在性能要求高的关键环境中使用的常见表达式。 这些表达方式目前无法编译：

* expressions involving assignment\(涉及转让的表达\)
* expressions relying on the conversion service\(依赖转换服务的表达式\)
* expressions using custom resolvers or accessors\(使用自定义解析器或访问器的表达式\)
* expressions using selection or projection\(使用选择或投影的表达式\)

越来越多的类型的表达式将在未来可编译。

