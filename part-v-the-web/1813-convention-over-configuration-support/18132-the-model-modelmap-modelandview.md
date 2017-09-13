### 18.13.2The Model ModelMap \(ModelAndView\)

The`ModelMap`class is essentially a glorified`Map`that can make adding objects that are to be displayed in \(or on\) a`View`adhere to a common naming convention. Consider the following`Controller`implementation; notice that objects are added to the`ModelAndView`without any associated name specified.

```java
public class DisplayShoppingCartController implements Controller {

	public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) {

		List cartItems = // get a List of CartItem objects
		User user = // get the User doing the shopping

		ModelAndView mav = new ModelAndView("displayShoppingCart"); <-- the logical view name

		mav.addObject(cartItems); <-- look ma, no name, just the object
		mav.addObject(user); <-- and again ma!

		return mav;
	}
}
```

The`ModelAndView`class uses a`ModelMap`class that is a custom`Map`implementation that automatically generates a key for an object when an object is added to it. The strategy for determining the name for an added object is, in the case of a scalar object such as`User`, to use the short class name of the object’s class. The following examples are names that are generated for scalar objects put into a`ModelMap`instance.

* An`x.y.User`instance added will have the name`user`generated.

* An`x.y.Registration`instance added will have the name`registration`generated.

* An`x.y.Foo`instance added will have the name`foo`generated.

* A`java.util.HashMap`instance added will have the name`hashMap`generated. You probably want to be explicit about the name in this case because`hashMap`is less than intuitive.

* Adding`null`will result in an`IllegalArgumentException`being thrown. If the object \(or objects\) that you are adding could be`null`, then you will also want to be explicit about the name.

**What, no automatic pluralization?**

Spring Web MVC’s convention-over-configuration support does not support automatic pluralization. That is, you cannot add a`List`of`Person`objects to a`ModelAndView`and have the generated name be`people`.

This decision was made after some debate, with the "Principle of Least Surprise" winning out in the end.

The strategy for generating a name after adding a`Set`or a`List`is to peek into the collection, take the short class name of the first object in the collection, and use that with`List`appended to the name. The same applies to arrays although with arrays it is not necessary to peek into the array contents. A few examples will make the semantics of name generation for collections clearer:

* An`x.y.User[]`array with zero or more`x.y.User`elements added will have the name`userList`generated.

* An`x.y.Foo[]`array with zero or more`x.y.User`elements added will have the name`fooList`generated.

* A`java.util.ArrayList`with one or more`x.y.User`elements added will have the name`userList`generated.

* A`java.util.HashSet`with one or more`x.y.Foo`elements added will have the name`fooList`generated.

* An_empty_`java.util.ArrayList`will not be added at all \(in effect, the`addObject(..)`call will essentially be a no-op\).



