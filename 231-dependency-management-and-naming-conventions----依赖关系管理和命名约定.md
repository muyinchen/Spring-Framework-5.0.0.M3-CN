### 2.3.1 Dependency Management and Naming Conventions    `依赖关系管理和命名约定`

Dependency management and dependency injection are different things. To get those nice features of Spring into your application (like dependency injection) you need to assemble all the libraries needed (jar files) and get them onto your classpath at runtime, and possibly at compile time. These dependencies are not virtual components that are injected, but physical resources in a file system (typically). The process of dependency management involves locating those resources, storing them and adding them to classpaths. Dependencies can be direct (e.g. my application depends on Spring at runtime), or indirect (e.g. my application depends on `commons-dbcp` which depends on `commons-pool`). The indirect dependencies are also known as "transitive" and it is those dependencies that are hardest to identify and manage.

If you are going to use Spring you need to get a copy of the jar libraries that comprise the pieces of Spring that you need. To make this easier Spring is packaged as a set of modules that separate the dependencies as much as possible, so for example if you don’t want to write a web application you don’t need the spring-web modules. To refer to Spring library modules in this guide we use a shorthand naming convention `spring-*` or `spring-*.jar,` where `*` represents the short name for the module (e.g. `spring-core`, `spring-webmvc`, `spring-jms`, etc.). The actual jar file name that you use is normally the module name concatenated with the version number (e.g. *spring-core-5.0.0.M3.jar*).

Each release of the Spring Framework will publish artifacts to the following places:

- Maven Central, which is the default repository that Maven queries, and does not require any special configuration to use. Many of the common libraries that Spring depends on also are available from Maven Central and a large section of the Spring community uses Maven for dependency management, so this is convenient for them. The names of the jars here are in the form `spring-*-.jar` and the Maven groupId is `org.springframework`.
- In a public Maven repository hosted specifically for Spring. In addition to the final GA releases, this repository also hosts development snapshots and milestones. The jar file names are in the same form as Maven Central, so this is a useful place to get development versions of Spring to use with other libraries deployed in Maven Central. This repository also contains a bundle distribution zip file that contains all Spring jars bundled together for easy download.

So the first thing you need to decide is how to manage your dependencies: we generally recommend the use of an automated system like Maven, Gradle or Ivy, but you can also do it manually by downloading all the jars yourself.

You will find bellow the list of Spring artifacts. For a more complete description of each modules, see [Section 2.2, “Modules”](http://docs.spring.io/spring/docs/5.0.0.M3/spring-framework-reference/htmlsingle/#overview-modules).

依赖管理和依赖注入是不同的。为了让 `Spring `的这些不错的功能运用到运用程序中(比如依赖注入)，你需要导入所有需要的库（jar文件），并且在编译、运行的时候将它们放到你的类路径中。这些依赖关系不是注入的虚拟组件，而是文件系统中的物理资源（通常）。依赖关系管理的过程包括定位这些资源，存储它们并将它们添加到类路径。依赖可以是直接的（例如我的应用程序依赖于`Spring`在运行时）或间接（例如我的应用程序依赖于`commons-dbcp`，这取决于`commons-pool`）。间接依赖性也称为“传递性”，它是那些最难识别和管理的依赖性。

如果你要使用`Spring`，你需要得到一个包含你需要的`Spring`的`jar`库的`jar`文件。为了使这过程更简单，`Spring`被打包为一组模块，尽可能地分离依赖，所以假如你不想编写一个`web`应用程序，你不需要`spring-web`模块。为了在本指南中引用`Spring`库模块，我们使用一个简写命名约定`spring- *`或`spring - *.jar`，其中*表示模块的短名称（例如spring-core，spring-webmvc，spring-jms等）。 ）。你使用的实际jar文件名通常是与版本号连接的模块名（例如`spring-core-5.0.0.M3.jar`）。

`Spring Framework`的每个版本都会将工件发布到以下位置：

- `Maven Central`，它是`Maven`查询的默认存储库，不需要使用任何特殊配置。 `Spring`依赖的许多通用库也可以从`Maven Central`获得，`Spring`社区的一大部分使用`Maven`进行依赖关系管理，所以这对他们很方便。这里的jars的名字是在`spring - * - .jar`和`Maven groupId`是`org.springframework`。
- 在一个专门为`Spring`托管的公共`Maven`仓库中。除了最终`GA`版本，此存储库还托管开发快照和里程碑。 `jar`文件名的格式与`Maven Central`相同，因此这是一个有用的地方，可以让`Spring`的开发版本与`Maven Central`中部署的其他库一起使用。此存储库还包含捆绑包分发`zip`文件，其中包含捆绑在一起的所有`Spring jar`以便于下载。

所以你需要决定的第一件事是如何管理你的依赖：我们一般建议使用一个自动化系统，如`Maven，Gradle`或`Ivy`，但你也可以手动下载所有的`jar`本身。

你会发现下面的`Spring`工件列表。有关每个模块的更完整的描述，请参见第2.2节“模块”。

**表2.1。 Spring框架组件**

| GroupId             | ArtifactId               | Description                              |
| ------------------- | ------------------------ | ---------------------------------------- |
| org.springframework | spring-aop               | Proxy-based AOP support 基于代理的AOP支持       |
| org.springframework | spring-aspects           | AspectJ based aspects 基于AspectJ的切面       |
| org.springframework | spring-beans             | Beans support, including Groovy                               Bean支持，包括Groovy |
| org.springframework | spring-context           | Application context runtime, including scheduling and remoting abstractions  应用程序上下文运行时，包括调度和远程抽象 |
| org.springframework | spring-context-support   | Support classes for integrating common third-party libraries into a Spring application context  支持将常见的第三方库集成到Spring应用程序上下文中的类 |
| org.springframework | spring-core              | Core utilities, used by many other Spring modules  核心应用程序，由许多其他Spring模块使用 |
| org.springframework | spring-expression        | Spring Expression Language (SpEL)        |
| org.springframework | spring-instrument        | Instrumentation agent for JVM bootstrapping JVM引导的工具代理 |
| org.springframework | spring-instrument-tomcat | Instrumentation agent for Tomcat    Tomcat的工具代理 |
| org.springframework | spring-jdbc              | JDBC support package, including DataSource setup and JDBC access support    JDBC支持包，包括DataSource设置和JDBC访问支持 |
| org.springframework | spring-jms               | JMS support package, including helper classes to send and receive JMS messages    JMS支持包，包括用于发送和接收JMS消息的助手类 |
| org.springframework | spring-messaging         | Support for messaging architectures and protocols 支持消息架构和协议 |
| org.springframework | spring-orm               | Object/Relational Mapping, including JPA and Hibernate support    对象/关系映射，包括JPA和Hibernate支持 |
| org.springframework | spring-oxm               | Object/XML Mapping  对象/ XML映射            |
| org.springframework | spring-test              | Support for unit testing and integration testing Spring components  支持单元测试和集成测试的Spring组件 |
| org.springframework | spring-tx                | Transaction infrastructure, including DAO support and JCA integration 事务基础设施，包括DAO支持和集成制定 |
| org.springframework | spring-web               | Web support packages, including client and web remoting   Web支持包，包括客户端和Web远程处理 |
| org.springframework | spring-webmvc            | REST Web Services and model-view-controller implementation for web applications   Web应用程序的REST Web服务和模型 - 视图 - 控制器实现 |
| org.springframework | spring-websocket         | WebSocket and SockJS implementations, including STOMP support   WebSocket和SockJS实现，包括STOMP支持 |

#### Spring Dependencies and Depending on Spring  `Spring依赖管理`

Although Spring provides integration and support for a huge range of enterprise and other external tools, it intentionally keeps its mandatory dependencies to an absolute minimum: you shouldn’t have to locate and download (even automatically) a large number of jar libraries in order to use Spring for simple use cases. For basic dependency injection there is only one mandatory external dependency, and that is for logging (see below for a more detailed description of logging options).

Next we outline the basic steps needed to configure an application that depends on Spring, first with Maven and then with Gradle and finally using Ivy. In all cases, if anything is unclear, refer to the documentation of your dependency management system, or look at some sample code - Spring itself uses Gradle to manage dependencies when it is building, and our samples mostly use Gradle or Maven.

虽然`Spring`提供了大量对企业和其他外部工具的集成和支持，它有意地将其强制依赖保持在绝对最低限度：你无需查找和下载（甚至自动）大量的jar库，以便 使用`Spring`的简单用例。 对于基本依赖注入，只有一个强制性的外部依赖，这是用于日志记录的（有关日志选项的更详细描述，请参见下文）。

接下来，我们概述配置依赖于`Spring`的应用程序所需的基本步骤，首先使用`Maven`，然后使用`Gradle`，最后使用`Ivy`。 在所有情况下，如果有什么不清楚，请参考依赖管理系统的文档，或者查看一些示例代码 - `Spring`本身使用`Gradle`在构建时管理依赖关系，我们的示例主要使用`Gradle`或`Maven`。

#### Maven Dependency Management  `Maven 依赖管理`

If you are using [Maven](http://maven.apache.org/) for dependency management you don’t even need to supply the logging dependency explicitly. For example, to create an application context and use dependency injection to configure an application, your Maven dependencies will look like this:

如果你使用`Maven`进行依赖关系管理，你甚至不需要显式提供日志记录依赖关系。 例如，要创建应用程序上下文并使用依赖注入来配置应用程序，你的`Maven`依赖关系将如下所示：

```
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>5.0.0.M3</version>
        <scope>runtime</scope>
    </dependency>
</dependencies>
```

That’s it. Note the scope can be declared as runtime if you don’t need to compile against Spring APIs, which is typically the case for basic dependency injection use cases.

The example above works with the Maven Central repository. To use the Spring Maven repository (e.g. for milestones or developer snapshots), you need to specify the repository location in your Maven configuration. For full releases:

就是这样。 注意，如果不需要针对`Spring API`进行编译，范围(`scope` )可以声明为运行时(`runtime`)，这通常是基本依赖注入使用用例。

上面的示例使用`Maven Central`存储库。 要使用`Spring Maven`存储库（例如，用于里程碑或开发人员快照），你需要在`Maven`配置中指定存储库位置。 完整版本：

```
<repositories>
    <repository>
        <id>io.spring.repo.maven.release</id>
        <url>http://repo.spring.io/release/</url>
        <snapshots><enabled>false</enabled></snapshots>
    </repository>
</repositories>
```

里程碑：

```
<repositories>
    <repository>
        <id>io.spring.repo.maven.milestone</id>
        <url>http://repo.spring.io/milestone/</url>
        <snapshots><enabled>false</enabled></snapshots>
    </repository>
</repositories>
```

And for snapshots:  以及快照：

```
<repositories>
    <repository>
        <id>io.spring.repo.maven.snapshot</id>
        <url>http://repo.spring.io/snapshot/</url>
        <snapshots><enabled>true</enabled></snapshots>
    </repository>
</repositories>
```

#### Maven "Bill Of Materials" Dependency  

It is possible to accidentally mix different versions of Spring JARs when using Maven. For example, you may find that a third-party library, or another Spring project, pulls in a transitive dependency to an older release. If you forget to explicitly declare a direct dependency yourself, all sorts of unexpected issues can arise.

To overcome such problems Maven supports the concept of a "bill of materials" (BOM) dependency. You can import the `spring-framework-bom` in your `dependencyManagement` section to ensure that all spring dependencies (both direct and transitive) are at the same version.

在使用`Maven`时，可能会意外混合不同版本的`Spring JAR`。 例如，你可能会发现第三方库或另一个`Spring`项目将传递依赖项拉入旧版本。 如果你忘记自己显式声明一个直接依赖，可能会出现各种意想不到的问题。

为了克服这种问题，`Maven`支持“物料清单”（`BOM`）依赖的概念。 你可以在`dependencyManagement`部分中导入`spring-framework-bom`，以确保所有`spring`依赖项（直接和可传递）具有相同的版本。

```
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-framework-bom</artifactId>
            <version>5.0.0.M3</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

An added benefit of using the BOM is that you no longer need to specify the `<version>` attribute when depending on Spring Framework artifacts:

使用 `BOM `后，当依赖 `Spring Framework `组件后，无需再指定`<version>` 属性



```
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-web</artifactId>
    </dependency>
<dependencies>
```

#### Gradle Dependency Management  `通过Gradle 做依赖管理`

To use the Spring repository with the [Gradle](http://www.gradle.org/) build system, include the appropriate URL in the `repositories` section:

要将`Spring`存储库与`Gradle`构建系统一起使用，请在`repositories`部分中包含相应的`URL`：

```
repositories {
    mavenCentral()
    // and optionally...
    maven { url "http://repo.spring.io/release" }
}
```

You can change the `repositories` URL from `/release` to `/milestone` or `/snapshot` as appropriate. Once a repository has been configured, you can declare dependencies in the usual Gradle way:

你可以根据需要将 `repositories` 中URL从`/ release`更改为`/ milestone`或`/ snapshot`。 一旦配置了 `repositories`，就可以按照通常的`Gradle`方式声明依赖关系：

```
dependencies {
    compile("org.springframework:spring-context:5.0.0.M3")
    testCompile("org.springframework:spring-test:5.0.0.M3")
}
```

#### Ivy Dependency Management  ` Ivy 依赖管理`

If you prefer to use [Ivy](http://ant.apache.org/ivy) to manage dependencies then there are similar configuration options.

To configure Ivy to point to the Spring repository add the following resolver to your `ivysettings.xml`:

如果你喜欢使用[Ivy](http://ant.apache.org/ivy)来管理依赖，那么有类似的配置选项。

要配置`Ivy`指向`Spring`存储库，请将以下解析器添加到你的`ivysettings.xml`：

```
<resolvers>
    <ibiblio name="io.spring.repo.maven.release"
            m2compatible="true"
            root="http://repo.spring.io/release/"/>
</resolvers>
```

You can change the `root` URL from `/release/` to `/milestone/` or `/snapshot/` as appropriate.

Once configured, you can add dependencies in the usual way. For example (in `ivy.xml`):



你可以根据需要将`root`   URL从`/ release /`更改为`/ milestone /`或`/ snapshot /`。

配置后，你可以按照通常的方式添加依赖关系。 例如（在`ivy.xml`中）：

```
<dependency org="org.springframework"
    name="spring-core" rev="5.0.0.M3" conf="compile->runtime"/>
```

#### Distribution Zip Files

Although using a build system that supports dependency management is the recommended way to obtain the Spring Framework, it is still possible to download a distribution zip file.

Distribution zips are published to the Spring Maven Repository (this is just for our convenience, you don’t need Maven or any other build system in order to download them).

To download a distribution zip open a web browser to [http://repo.spring.io/release/org/springframework/spring](http://repo.spring.io/release/org/springframework/spring) and select the appropriate subfolder for the version that you want. Distribution files end `-dist.zip`, for example spring-framework-{spring-version}-RELEASE-dist.zip. Distributions are also published for [milestones](http://repo.spring.io/milestone/org/springframework/spring) and [snapshots](http://repo.spring.io/snapshot/org/springframework/spring).



尽管使用支持依赖性管理的构建系统是获取`Spring Framework`的推荐方式，但仍然可以下载分发`zip`文件。

分发`zip`是发布到`Spring Maven`仓库（这只是为了我们的方便，你不需要`Maven`或任何其他构建系统为了下载它们）。

要下载分发`zip`，请打开Web浏览器到http://repo.spring.io/release/org/springframework/spring，然后为所需的版本选择适当的子文件夹。 分发文件结尾是 `-dist.zip`，例如`spring-framework- {spring-version} -RELEASE-dist.zip`。 还分发了里程碑和快照的分发。