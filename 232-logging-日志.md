### 2.3.2 Logging `日志`

Logging is a very important dependency for Spring because *a)* it is the only mandatory external dependency, *b)* everyone likes to see some output from the tools they are using, and *c)* Spring integrates with lots of other tools all of which have also made a choice of logging dependency. One of the goals of an application developer is often to have unified logging configured in a central place for the whole application, including all external components. This is more difficult than it might have been since there are so many choices of logging framework.

The mandatory logging dependency in Spring is the Jakarta Commons Logging API (JCL). We compile against JCL and we also make JCL `Log` objects visible for classes that extend the Spring Framework. It’s important to users that all versions of Spring use the same logging library: migration is easy because backwards compatibility is preserved even with applications that extend Spring. The way we do this is to make one of the modules in Spring depend explicitly on `commons-logging`(the canonical implementation of JCL), and then make all the other modules depend on that at compile time. If you are using Maven for example, and wondering where you picked up the dependency on `commons-logging`, then it is from Spring and specifically from the central module called `spring-core`.

The nice thing about `commons-logging` is that you don’t need anything else to make your application work. It has a runtime discovery algorithm that looks for other logging frameworks in well known places on the classpath and uses one that it thinks is appropriate (or you can tell it which one if you need to). If nothing else is available you get pretty nice looking logs just from the JDK (java.util.logging or JUL for short). You should find that your Spring application works and logs happily to the console out of the box in most situations, and that’s important.



日志记录是`Spring`的一个非常重要的依赖，因为`a）`它是唯一的强制性的外部依赖，`b）`每个人都喜欢从他们使用的工具看到一些输出，和`c）`Spring集成了许多其他工具，日志依赖性的选择。应用程序开发人员的目标之一通常是在整个应用程序的中心位置配置统一日志记录，包括所有外部组件。这就更加困难，因为它可能已经有太多选择的日志框架。

`Spring`中的强制性日志依赖性是`Jakarta Commons Logging API（JCL）`。我们编译`JCL`，我们也使`JCL Log`对象对于扩展`Spring`框架的类可见。对于用户来说，所有版本的Spring都使用相同的日志库很重要：迁移很容易，因为即使使用扩展`Spring`的应用程序也保持向后兼容性。我们这样做的方式是使Spring中的一个模块显式地依赖commons-logging（JCL的规范实现），然后使所有其他模块在编译时依赖它。如果你使用Maven为例，并想知道你在哪里选择对`commons-logging`的依赖，那么它是从`Spring`，特别是从中央模块称为`spring-core`(`关于此处，理解就好，翻译的不到位`)。

关于`commons-logging`的好处是，你不需要任何其他东西来就能让你的应用程序工作。它有一个运行时发现算法，该算法在众所周知的`classpath`路径下寻找其他日志框架，并使用它认为是合适的（或者你可以告诉它，如果你需要）。如果没有其他可用的，你可以从`JDK`（`java.util.logging`或简称`JUL`）获得漂亮的查看日志。你应该会发现，你的`Spring`应用程序在大多数情况下可以很好地工作和记录到控制台，这很重要。

#### Not Using Commons Logging  `不使用 Commons Logging`

Unfortunately, the runtime discovery algorithm in `commons-logging`, while convenient for the end-user, is problematic. If we could turn back the clock and start Spring now as a new project it would use a different logging dependency. The first choice would probably be the Simple Logging Facade for Java ( [SLF4J](http://www.slf4j.org/)), which is also used by a lot of other tools that people use with Spring inside their applications.

There are basically two ways to switch off `commons-logging`:

1. Exclude the dependency from the `spring-core` module (as it is the only module that explicitly depends on `commons-logging`)
2. Depend on a special `commons-logging` dependency that replaces the library with an empty jar (more details can be found in the [SLF4J FAQ](http://slf4j.org/faq.html#excludingJCL))

To exclude commons-logging, add the following to your `dependencyManagement` section:



不幸的是，公共日志中的运行时发现算法虽然对于终端用户方便，但是是有问题的。 如果我们可以时光倒流并启动`Spring`作为一个新项目，它将使用不同的日志依赖关系。 第一个选择可能是用于`Java`的简单日志外观（[SLF4J](http://www.slf4j.org/)），它也被许多其他工具用于`Spring`在其应用程序中使用。

基本上有两种方法关闭`commons-logging`：

1.从`spring-core`模块中排除依赖性（因为它是唯一显式依赖`commons-logging`的模块）
2.依赖于一个特殊的`commons-logging`依赖，用一个空jar替换库（更多细节可以在 [SLF4J FAQ](http://slf4j.org/faq.html#excludingJCL)中找到）

要排除`commons-logging`，请将以下内容添加到你的`dependencyManagement`部分：

```
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
        <version>5.0.0.M4</version>
        <exclusions>
            <exclusion>
                <groupId>commons-logging</groupId>
                <artifactId>commons-logging</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
</dependencies>
```

Now this application is probably broken because there is no implementation of the JCL API on the classpath, so to fix it a new one has to be provided. In the next section we show you how to provide an alternative implementation of JCL using SLF4J as an example.

现在这个应用程序运行不了，因为没有在类路径上实现`JCL API`，所以要解决它需要提供一个新的。 在下一节中，我们将向你展示如何使用`SLF4J`提供`JCL`的替代实现。

#### Using SLF4J  使用 SLF4J

SLF4J is a cleaner dependency and more efficient at runtime than `commons-logging` because it uses compile-time bindings instead of runtime discovery of the other logging frameworks it integrates. This also means that you have to be more explicit about what you want to happen at runtime, and declare it or configure it accordingly. SLF4J provides bindings to many common logging frameworks, so you can usually choose one that you already use, and bind to that for configuration and management.

SLF4J provides bindings to many common logging frameworks, including JCL, and it also does the reverse: bridges between other logging frameworks and itself. So to use SLF4J with Spring you need to replace the `commons-logging` dependency with the SLF4J-JCL bridge. Once you have done that then logging calls from within Spring will be translated into logging calls to the SLF4J API, so if other libraries in your application use that API, then you have a single place to configure and manage logging.

A common choice might be to bridge Spring to SLF4J, and then provide explicit binding from SLF4J to Log4J. You need to supply 4 dependencies (and exclude the existing `commons-logging`): the bridge, the SLF4J API, the binding to Log4J, and the Log4J implementation itself. In Maven you would do that like this

`SLF4J`是一个更简洁的依赖，在运行时比`commons-logging`更高效，因为它使用编译时绑定，而不是其集成的其他日志框架的运行时发现。这也意味着你必须更明确地了解你想在运行时发生什么，并声明它或相应地配置它。 `SLF4J`提供了对许多常见日志框架的绑定，因此你通常可以选择一个已经使用的绑定，并绑定到配置和管理。

`SLF4J`提供对许多常见日志框架（包括`JCL`）的绑定，并且它也做了反向工作：充当其他日志框架与其自身之间的桥梁。因此，要在Spring中使用`SLF4J`，需要使用`SLF4J-JCL`桥替换`commons-logging`依赖关系。一旦你这样做，那么来自`Spring`内部的日志调用将被转换为对`SLF4J API`的日志调用，因此如果应用程序中的其他库使用该API，那么你有一个地方可以配置和管理日志记录。

常见的选择可能是将`Spring`桥接到`SLF4J`，然后提供从`SLF4J`到`Log4J`的显式绑定。你需要提供4个依赖（并排除现有的`commons-logging`）：桥梁，`SLF4J API`，绑定到`Log4J`和`Log4J`实现本身。在`Maven`，你会这样做

```
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
        <version>5.0.0.M4</version>
        <exclusions>
            <exclusion>
                <groupId>commons-logging</groupId>
                <artifactId>commons-logging</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>jcl-over-slf4j</artifactId>
        <version>1.5.8</version>
    </dependency>
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-api</artifactId>
        <version>1.5.8</version>
    </dependency>
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-log4j12</artifactId>
        <version>1.5.8</version>
    </dependency>
    <dependency>
        <groupId>log4j</groupId>
        <artifactId>log4j</artifactId>
        <version>1.2.14</version>
    </dependency>
</dependencies>
```

That might seem like a lot of dependencies just to get some logging. Well it is, but it *is* optional, and it should behave better than the vanilla `commons-logging` with respect to classloader issues, notably if you are in a strict container like an OSGi platform. Allegedly there is also a performance benefit because the bindings are at compile-time not runtime.

A more common choice amongst SLF4J users, which uses fewer steps and generates fewer dependencies, is to bind directly to [Logback](http://logback.qos.ch/). This removes the extra binding step because Logback implements SLF4J directly, so you only need to depend on two libraries not four ( `jcl-over-slf4j` and `logback`). If you do that you might also need to exclude the slf4j-api dependency from other external dependencies (not Spring), because you only want one version of that API on the classpath.

这可能看起来像很多依赖只是为了获得一些日志。 它的确如此，但它是可选的，它在关于类加载器的问题上应该比 `commons-logging` 表现的更加的好，特别是当它运行在在一个严格的容器中像`OSGi `平台。 据称，还有一个性能优势，因为绑定是在编译时，而不是运行时。

`SLF4J`用户中更常见的选择是使用较少的步骤和生成较少的依赖关系，它是直接绑定到`Logback`。 这消除了额外的绑定步骤，因为`Logback`直接实现`SLF4J`，所以你只需要依赖于两个不是四个库（`jcl-over-slf4j`和`logback`）。 如果你这样做，你可能还需要从其他外部依赖（不是`Spring`）中排除`slf4j-api`依赖，因为你只需要在类路径上有一个版本的`API`。

#### Using Log4J

Many people use [Log4j](http://logging.apache.org/log4j) as a logging framework for configuration and management purposes. It’s efficient and well-established, and in fact it’s what we use at runtime when we build and test Spring. Spring also provides some utilities for configuring and initializing Log4j, so it has an optional compile-time dependency on Log4j in some modules.

To make Log4j work with the default JCL dependency ( `commons-logging`) all you need to do is put Log4j on the classpath, and provide it with a configuration file (`log4j.properties` or `log4j.xml` in the root of the classpath). So for Maven users this is your dependency declaration:



许多人使用[Log4j](http://logging.apache.org/log4j)作为日志框架用于配置和管理目的。 它是高效的和成熟的，事实上，这是我们在运行时使用时，我们构建和测试`Spring`。 `Spring`还提供了一些用于配置和初始化`Log4j`的实用程序，所以它在一些模块中对`Log4j`有一个可选的编译时依赖。

要使`Log4j`使用默认的`JCL`依赖（`commons-logging`），所有你需要做的是将`Log4j`放在类路径上，并为它提供一个配置文件（在类路径的根目录下的`log4j.properties`或`log4j.xml`）。 所以对于`Maven`用户，这是你的依赖性声明：



```
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
        <version>5.0.0.M4</version>
    </dependency>
    <dependency>
        <groupId>log4j</groupId>
        <artifactId>log4j</artifactId>
        <version>1.2.14</version>
    </dependency>
</dependencies>
```

And here’s a sample log4j.properties for logging to the console:

下面是一个简单的` log4j.properties` 的实例，用于将日志打印到控制台：



```
log4j.rootCategory=INFO, stdout

log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%d{ABSOLUTE} %5p %t %c{2}:%L - %m%n

log4j.category.org.springframework.beans.factory=DEBUG
```

##### Runtime Containers with Native JCL

Many people run their Spring applications in a container that itself provides an implementation of JCL. IBM Websphere Application Server (WAS) is the archetype. This often causes problems, and unfortunately there is no silver bullet solution; simply excluding `commons-logging` from your application is not enough in most situations.

To be clear about this: the problems reported are usually not with JCL per se, or even with `commons-logging`: rather they are to do with binding `commons-logging` to another framework (often Log4J). This can fail because `commons-logging` changed the way they do the runtime discovery in between the older versions (1.0) found in some containers and the modern versions that most people use now (1.1). Spring does not use any unusual parts of the JCL API, so nothing breaks there, but as soon as Spring or your application tries to do any logging you can find that the bindings to Log4J are not working.

In such cases with WAS the easiest thing to do is to invert the class loader hierarchy (IBM calls it "parent last") so that the application controls the JCL dependency, not the container. That option isn’t always open, but there are plenty of other suggestions in the public domain for alternative approaches, and your mileage may vary depending on the exact version and feature set of the container.



许多人在一个容器中运行他们的`Spring`应用程序，该容器本身提供了`JCL`的实现。 `IBM Websphere`应用服务器（`WAS`）就是一个例子。这常常导致问题，不幸的是没有一个一劳永逸解决方案;在大多数情况下，只是从你的应用程序中排除`commons-logging`是不够的。

要明确这一点：报告的问题通常不是`JCL`本身，或者甚至与`commons-logging`：而是他们要绑定到另一个框架（通常Log4J）的公共日志。这可能会失败，因为`commons-logging`改变了在一些容器中发现的旧版本（1.0）和大多数人现在使用的现代版本（1.1）之间执行运行时发现的方式。 `Spring`不使用`JCL API`的任何不常见的模块，所以没有什么问题出现，但一旦`Spring`或你的应用程序试图做任何日志记录，你可以发现绑定的`Log4J`不工作了。

在这种情况下，使用`WAS`，最容易做的事情是反转类加载器层次结构（IBM称之为“父最后一个”），以便应用程序控制`JCL`依赖关系，而不是容器。该选项并不总是开放的，但在公共领域有许多其他建议替代方法，你的里程可能会根据容器的确切版本和功能集而有所不同。