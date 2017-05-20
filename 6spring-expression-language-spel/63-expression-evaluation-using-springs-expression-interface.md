## 6.3 使用Spring表达式接口的表达式评估

本节介绍了SpEL接口及其表达式语言的简单使用。完整的语言参考可以在“语言参考”一节中找到。

以下代码介绍了SpEL API来评估文字字符串表达式“Hello World”。

```java
ExpressionParser parser = new SpelExpressionParser();
Expression exp = parser.parseExpression("'Hello World'");
String message = (String) exp.getValue();
```

变量message的值只是 'Hello World'.

您最有可能使用的SpEL类和接口位于包 `org.springframework.expression` 及其子包和包`spel.support`

接口`ExpressionParser`负责解析表达式字符串。 在此示例中，表达式字符串是由周围的单引号表示的字符串文字。接口`Expression`负责评估先前定义的表达式字符串。 当分别调用`parser.parseExpression`和`exp.getValue`时，有两个可以抛出的异常，`ParseException`和`EvaluationException`。

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

Spel的更常见的用法是提供一个针对特定对象实例（称为根对象）进行评估的表达式字符串。 这里有两个选项 ，并且选择哪个由反对当前被验证的表示式的对象是否在每次调用后而改变再验证表达式来决定。在以下示例中，我们从`Inventor`类的实例中检索`name`属性。

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

在最后一行，字符串变量name的值将被设置为“Nikola Tesla”。 `StandardEvaluationContext`类可以指定哪个对象的“name”属性将被评估。 如果根对象不太可能改变，这是使用的机制，可以在评估上下文中简单地设置一次。 如果根对象可能会重复更改，则可以在每次调用getValue时提供该对象，如下例所示：

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

在这种情况下，目标对象`tesla`已经直接提供给`getValue`，表达式评估基础动作即在内部创建和管理默认评估的上下文 - 它不需要提供。

The StandardEvaluationContext is relatively expensive to construct and during repeated usage it builds up cached state that enables subsequent expression evaluations to be performed more quickly. For this reason it is better to cache and reuse them where possible, rather than construct a new one for each expression evaluation.

In some cases it can be desirable to use a configured evaluation context and yet still supply a different root object on each call to `getValue`. `getValue` allows both to be specified on the same call. In these situations the root object passed on the call is considered to override any \(which maybe null\) specified on the evaluation context.

| ![\[Note\]](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| --- |
| In standalone usage of SpEL there is a need to create the parser, parse expressions and perhaps provide evaluation contexts and a root context object. However, more common usage is to provide only the SpEL expression string as part of a configuration file, for example for Spring bean or Spring Web Flow definitions. In this case, the parser, evaluation context, root object and any predefined variables are all set up implicitly, requiring the user to specify nothing other than the expressions. |

As a final introductory example, the use of a boolean operator is shown using the Inventor object in the previous example.

```java
Expression exp = parser.parseExpression("name == 'Nikola Tesla'");
boolean result = exp.getValue(context, Boolean.class); // evaluates to true
```

### 6.3.1 The EvaluationContext interface

The interface `EvaluationContext` is used when evaluating an expression to resolve properties, methods, fields, and to help perform type conversion. The out-of-the-box implementation, `StandardEvaluationContext`, uses reflection to manipulate the object, caching `java.lang.reflect.Method`, `java.lang.reflect.Field`, and `java.lang.reflect.Constructor` instances for increased performance.

The `StandardEvaluationContext` is where you may specify the root object to evaluate against via the method `setRootObject()` or passing the root object into the constructor. You can also specify variables and functions that will be used in the expression using the methods `setVariable()` and `registerFunction()`. The use of variables and functions are described in the language reference sections [Variables](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#expressions-ref-variables) and [Functions](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#expressions-ref-functions). The `StandardEvaluationContext` is also where you can register custom `ConstructorResolver`s, `MethodResolver`s, and `PropertyAccessor`s to extend how SpEL evaluates expressions. Please refer to the JavaDoc of these classes for more details.

#### Type Conversion

By default SpEL uses the conversion service available in Spring core \( `org.springframework.core.convert.ConversionService`\). This conversion service comes with many converters built in for common conversions but is also fully extensible so custom conversions between types can be added. Additionally it has the key capability that it is generics aware. This means that when working with generic types in expressions, SpEL will attempt conversions to maintain type correctness for any objects it encounters.

What does this mean in practice? Suppose assignment, using `setValue()`, is being used to set a `List` property. The type of the property is actually `List<Boolean>`. SpEL will recognize that the elements of the list need to be converted to `Boolean` before being placed in it. A simple example:

```java
class Simple {
    public List<Boolean> booleanList = new ArrayList<Boolean>();
}

Simple simple = new Simple();

simple.booleanList.add(true);

StandardEvaluationContext simpleContext = new StandardEvaluationContext(simple);

// false is passed in here as a string. SpEL and the conversion service will
// correctly recognize that it needs to be a Boolean and convert it
parser.parseExpression("booleanList[0]").setValue(simpleContext, "false");

// b will be false
Boolean b = simple.booleanList.get(0);
```

### 6.3.2 Parser configuration

It is possible to configure the SpEL expression parser using a parser configuration object \(`org.springframework.expression.spel.SpelParserConfiguration`\). The configuration object controls the behavior of some of the expression components. For example, if indexing into an array or collection and the element at the specified index is `null` it is possible to automatically create the element. This is useful when using expressions made up of a chain of property references. If indexing into an array or list and specifying an index that is beyond the end of the current size of the array or list it is possible to automatically grow the array or list to accommodate that index.

```java
class Demo {
    public List<String> list;
}

// Turn on:
// - auto null reference initialization
// - auto collection growing
SpelParserConfiguration config = new SpelParserConfiguration(true,true);

ExpressionParser parser = new SpelExpressionParser(config);

Expression expression = parser.parseExpression("list[3]");

Demo demo = new Demo();

Object o = expression.getValue(demo);

// demo.list will now be a real collection of 4 entries
// Each entry is a new empty String
```

It is also possible to configure the behaviour of the SpEL expression compiler.

### 6.3.3 SpEL compilation

Spring Framework 4.1 includes a basic expression compiler. Expressions are usually interpreted which provides a lot of dynamic flexibility during evaluation but does not provide the optimum performance. For occasional expression usage this is fine, but when used by other components like Spring Integration, performance can be very important and there is no real need for the dynamism.

The new SpEL compiler is intended to address this need. The compiler will generate a real Java class on the fly during evaluation that embodies the expression behavior and use that to achieve much faster expression evaluation. Due to the lack of typing around expressions the compiler uses information gathered during the interpreted evaluations of an expression when performing compilation. For example, it does not know the type of a property reference purely from the expression but during the first interpreted evaluation it will find out what it is. Of course, basing the compilation on this information could cause trouble later if the types of the various expression elements change over time. For this reason compilation is best suited to expressions whose type information is not going to change on repeated evaluations.

For a basic expression like this:

`someArray[0].someProperty.someOtherProperty < 0.1`

which involves array access, some property derefencing and numeric operations, the performance gain can be very noticeable. In an example micro benchmark run of 50000 iterations, it was taking 75ms to evaluate using only the interpreter and just 3ms using the compiled version of the expression.

#### Compiler configuration

The compiler is not turned on by default, but there are two ways to turn it on. It can be turned on using the parser configuration process discussed earlier or via a system property when SpEL usage is embedded inside another component. This section discusses both of these options.

Is is important to understand that there are a few modes the compiler can operate in, captured in an enum \(`org.springframework.expression.spel.SpelCompilerMode`\). The modes are as follows:

* `OFF` - The compiler is switched off; this is the default.
* `IMMEDIATE` - In immediate mode the expressions are compiled as soon as possible. This is typically after the first interpreted evaluation. If the compiled expression fails \(typically due to a type changing, as described above\) then the caller of the expression evaluation will receive an exception.
* `MIXED` - In mixed mode the expressions silently switch between interpreted and compiled mode over time. After some number of interpreted runs they will switch to compiled form and if something goes wrong with the compiled form \(like a type changing, as described above\) then the expression will automatically switch back to interpreted form again. Sometime later it may generate another compiled form and switch to it. Basically the exception that the user gets in `IMMEDIATE` mode is instead handled internally.

`IMMEDIATE` mode exists because `MIXED` mode could cause issues for expressions that have side effects. If a compiled expression blows up after partially succeeding it may have already done something that has affected the state of the system. If this has happened the caller may not want it to silently re-run in interpreted mode since part of the expression may be running twice.

After selecting a mode, use the `SpelParserConfiguration` to configure the parser:

```java
SpelParserConfiguration config = new SpelParserConfiguration(SpelCompilerMode.IMMEDIATE,
    this.getClass().getClassLoader());

SpelExpressionParser parser = new SpelExpressionParser(config);

Expression expr = parser.parseExpression("payload");

MyMessage message = new MyMessage();

Object payload = expr.getValue(message);
```

When specifying the compiler mode it is also possible to specify a classloader \(passing null is allowed\). Compiled expressions will be defined in a child classloader created under any that is supplied. It is important to ensure if a classloader is specified it can see all the types involved in the expression evaluation process. If none is specified then a default classloader will be used \(typically the context classloader for the thread that is running during expression evaluation\).

The second way to configure the compiler is for use when SpEL is embedded inside some other component and it may not be possible to configure via a configuration object. In these cases it is possible to use a system property. The property `spring.expression.compiler.mode` can be set to one of the `SpelCompilerMode` enum values \(`off`, `immediate`, or `mixed`\).

#### Compiler limitations

With Spring Framework 4.1 the basic compilation framework is in place. However, the framework does not yet support compiling every kind of expression. The initial focus has been on the common expressions that are likely to be used in performance critical contexts. These kinds of expression cannot be compiled at the moment:

* expressions involving assignment
* expressions relying on the conversion service
* expressions using custom resolvers or accessors
* expressions using selection or projection

More and more types of expression will be compilable in the future.

