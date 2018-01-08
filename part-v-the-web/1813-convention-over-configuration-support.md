## 18.13 关于配置支持的公约

For a lot of projects, sticking to established conventions and having reasonable defaults is just what they \(the projects\) need, and Spring Web MVC now has explicit support for_convention over configuration_. What this means is that if you establish a set of naming conventions and suchlike, you can\_substantially\_cut down on the amount of configuration that is required to set up handler mappings, view resolvers,`ModelAndView`instances, etc. This is a great boon with regards to rapid prototyping, and can also lend a degree of \(always good-to-have\) consistency across a codebase should you choose to move forward with it into production.

Convention-over-configuration support addresses the three core areas of MVC: models, views, and controllers.

对于很多项目来说，坚持已有的约定和合理的默认值就是他们（项目）需要的东西，而Spring Web MVC现在已经明确支持配置公约。 这意味着如果你建立了一组命名约定等类似的东西，你可以大大减少设置处理程序映射，查看解析器，`ModelAndView`实例等所需的配置数量。这对于快速 原型设计，还可以在整个代码库中提供一定程度的（始终具有良好的一致性），如果您选择将其投入生产。

Convention-over-configuration支持解决了MVC的三个核心领域：模型，视图和控制器。

对于很多项目，坚持既定的约定和合理的默认值就是它们（项目）所需要的，而Spring Web MVC现在已经明确地支持_约定的配置_。这意味着如果您建立了一组命名约定等等，您可以_大幅度_减少设置处理程序映射，查看解析器，`ModelAndView`实例等所需的配置量 。这对于快速原型，并且如果您选择将其推向生产，还可以在代码库中提供一定程度的（始终如一的）一致性。

公约超配置支持解决了MVC的三个核心领域：模型，视图和控制器。

