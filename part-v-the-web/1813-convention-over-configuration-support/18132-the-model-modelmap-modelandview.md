### 18.13.2Model ModelMap \(ModelAndView\)

`ModelMap`类本质上是一个荣耀的`Map`，它可以使添加的对象在`View`中（或在其上）显示，并遵循一个通用的命名约定。 考虑下面的控制器实现; 注意到对象被添加到`ModelAndView`而没有指定任何关联的名字。

```java
public class DisplayShoppingCartController implements Controller {

    public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) {

        List cartItems = // 获取CartItem对象的列表
        User user = // 让用户进行购物

        ModelAndView mav = new ModelAndView("displayShoppingCart"); <-- 逻辑视图名称

        mav.addObject(cartItems); <-- 看ma，没有名字，只是对象
        mav.addObject(user); <-- and again ma!

        return mav;
    }
}
```

`ModelAndView`类使用一个`ModelMap`类，它是一个自定义的`Map`实现，当一个对象被添加到它时，该实现自动为一个对象生成一个键。 在像`User`这样的标量对象的情况下，为添加对象确定名称的策略是使用对象类的简短类名称。 以下示例是为放置在`ModelMap`实例中的标量对象生成的名称。

* `x.y.User`添加 的实例将生成名称`user`。

* `x.y.Registration`添加 的实例将生成名称`registration`。

* `x.y.Foo`添加 的实例将生成名称`foo`。

* `java.util.HashMap`添加的实例将生成名称`hashMap`。 你可能想在这种情况下明确名称，因为`hashMap`不是直观的。

* 添加`null`将导致引发`IllegalArgumentException`。 如果要添加的对象（或多个对象）可能为`null`，那么您还需要明确名称。

**什么，没有自动多元化？**

Spring Web MVC的常规配置支持支持不支持自动多元化。 也就是说，不能将一个`List`的`Person`对象列表添加到`ModelAndView`中，并将生成的名称作为`people`。

这个决定是经过一番辩论后做出来的，最后以“最低惊奇原则”获胜。

添加一个`Set`或者一个`List`之后生成一个名字的策略就是查看这个集合，获取这个集合中第一个对象的简短类名，并且在名字后面加上`List`。 对于数组也是如此，但是对于数组，不需要查看数组的内容。 一些例子会使集合的名字生成的语义更清晰：

* 添加了零个或多个`x.y.User`元素的`x.y.User[]`数组将生成名称`userList`。

* 添加了零个或多个`x.y.User`元素的`x.y.Foo[]`数组将会生成名称`fooList`。

* 添加了一个或多个`x.y.User`元素的`java.util.ArrayList`将生成的名称`userList`。

* 添加了一个或多个`x.y.Foo`元素的`java.util.HashSet`将具有生成的名称`fooList`。

* 根本不会添加 一个_空_`java.util.ArrayList`（实际上，`addObject(..)`调用本质上是无操作的）。



