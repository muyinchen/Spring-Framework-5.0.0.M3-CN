## 18.6 使用flash属性

Flash属性提供了一种请求来存储供另一个使用的属性的方法。 重定向时这是最常用的，例如，Post/Redirect/Get模式。 在重定向（通常在会话中）之前，Flash属性会临时保存，以便在重定向之后立即将其删除。

Spring MVC有两个支持flash属性的主要抽象。`FlashMap`用于保存Flash属性，而`FlashMapManager`用于存储，检索和管理`FlashMap`实例。

Flash属性支持始终处于“打开”状态，不需要显式启用即使未使用也不会导致创建HTTP会话。 在每个请求中，都有一个“输入”`FlashMap`，其中包含从前一个请求（如果有）传递的属性和具有属性的“输出”`FlashMap`，用于保存后续请求。 两个`FlashMap`实例都可以通过`RequestContextUtils`中的静态方法从Spring MVC中的任何地方访问。

带注解的控制器通常不需要直接使用`FlashMap`。 相反，`@RequestMapping`方法可以接受类型为`RedirectAttributes`的参数，并使用它为重定向场景添加Flash属性。 通过`RedirectAttributes`添加的`Flash`属性会自动传播到“输出”`FlashMap`。 同样，在重定向之后，来自“输入”`FlashMap`的属性将自动添加到提供目标URL的控制器的`Model`中。

**匹配请求到Flash属性**

Flash属性的概念存在于许多其他Web框架中，并且已经被证明有时会暴露给并发问题。 这是因为根据定义，闪存属性将被存储直到下一个请求。 然而，“下一个”请求可能不是预期的接收者，而是另一个异步请求（例如轮询或资源请求），在这种情况下，flash属性过早地被删除。

为了减少这种问题的可能性，`RedirectView`使用目标重定向URL的路径和查询参数自动“标记”`FlashMap`实例。 在查找“输入”`FlashMap`时，默认的`FlashMapManager`将该信息与传入的请求进行匹配。

这不能完全消除并发问题的可能性，但是仍然可以通过重定向URL中已经提供的信息来大大减少它。因此，使用Flash属性主要用于重定向方案。

