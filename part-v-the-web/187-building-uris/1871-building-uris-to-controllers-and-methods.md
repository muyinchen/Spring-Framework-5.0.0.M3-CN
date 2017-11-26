### 18.7.1构建控制器和方法的URI

Spring MVC还提供了一种构建控制器方法链接的机制。例如，给出：

```java
@Controller
@RequestMapping("/hotels/{hotel}")
public class BookingController {

    @GetMapping("/bookings/{booking}")
    public String getBooking(@PathVariable Long booking) {

    // ...
       }
}
```

您可以通过名称参考方法来准备链接：

```java
UriComponents uriComponents = MvcUriComponentsBuilder
    .fromMethodName(BookingController.class, "getBooking", 21).buildAndExpand(42);

URI uri = uriComponents.encode().toUri();
```

在上面的例子中，我们提供了实际的方法参数值，在这种情况下是long值21，被用作路径变量并被插入到URL中。 此外，我们提供了值42以填充任何剩余的URI变量，例如从类型级别请求映射继承的“hotel”变量。 如果方法有更多的参数，你可以为URL提供不需要的参数。 通常，只有`@PathVariable`和`@RequestParam`参数与构造URL相关。

还有其他方法可以使用`MvcUriComponentsBuilder`。 例如，您可以使用类似于通过代理进行模拟测试的技术，以避免通过名称引用控制器方法（该示例假定为`MvcUriComponentsBuilder.on`的静态导入）：

```java
UriComponents uriComponents = MvcUriComponentsBuilder
    .fromMethodCall(on(BookingController.class).getBooking(21)).buildAndExpand(42);

URI uri = uriComponents.encode().toUri();
```

上面的例子在`MvcUriComponentsBuilder`中使用静态方法。 在内部，他们依靠`ServletUriComponentsBuilder`从当前请求的方案，主机，端口，上下文路径和servlet路径中准备基本URL。 这在大多数情况下运作良好，但有时可能不足。 例如，您可能在请求的上下文之外（例如，准备链接的批处理过程），或者您可能需要插入路径前缀（例如，从请求路径中删除并需要重新插入链接的区域设置前缀）。

对于这种情况，您可以使用接受`UriComponentsBuilder`的静态“fromXxx”重载方法来使用基本URL。 或者，您可以使用基本URL创建`MvcUriComponentsBuilder`实例，然后使用基于实例的“withXxx”方法。 例如：

```java
UriComponentsBuilder base = ServletUriComponentsBuilder.fromCurrentContextPath().path("/en");
MvcUriComponentsBuilder builder = MvcUriComponentsBuilder.relativeTo(base);
builder.withMethodCall(on(BookingController.class).getBooking(21)).buildAndExpand(42);

URI uri = uriComponents.encode().toUri();
```



