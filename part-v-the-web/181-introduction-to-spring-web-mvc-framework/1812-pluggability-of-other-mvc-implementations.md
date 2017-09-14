### 18.1.2 其他MVC实现的可插拔性

对于某些项目，非Spring MVC实现更为可取。许多团队希望利用他们现有的技能和工具投资，例如使用JSF。

如果您不想使用Spring的Web MVC，但打算利用Spring提供的其他解决方案，您可以轻松地将您选择的Web MVC框架与Spring集成。通过其`ContextLoaderListener`简单地启动一个Spring根应用程序上下文，并通过任何动作对象中的`ServletContext`属性（或Spring的各自的帮助方法）访问它。没有涉及“插件”，因此不需要专门的集成。从Web层的角度来看，您只需使用Spring作为库，将根应用程序上下文实例作为入口点。

即使没有Spring的Web MVC，您的注册bean和Spring的服务也可以在您的指尖。在这种情况下，Spring不会与其他Web框架竞争。它简单地解决了纯Web MVC框架从bean配置到数据访问和事务处理的许多方面。所以您可以使用Spring中间层和/或数据访问层来丰富您的应用程序，即使您只想使用JDBC或Hibernate的事务抽象。

