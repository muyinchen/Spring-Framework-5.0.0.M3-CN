## 8.3 Advice API in Spring

Let’s now look at how Spring AOP handles advice.

### 8.3.1 Advice lifecycles

Each advice is a Spring bean. An advice instance can be shared across all advised objects, or unique to each advised object. This corresponds to *per-class* or *per-instance* advice.

Per-class advice is used most often. It is appropriate for generic advice such as transaction advisors. These do not depend on the state of the proxied object or add new state; they merely act on the method and arguments.

Per-instance advice is appropriate for introductions, to support mixins. In this case, the advice adds state to the proxied object.

It’s possible to use a mix of shared and per-instance advice in the same AOP proxy.

### 8.3.2 Advice types in Spring

Spring provides several advice types out of the box, and is extensible to support arbitrary advice types. Let us look at the basic concepts and standard advice types.

#### Interception around advice

The most fundamental advice type in Spring is *interception around advice*.

Spring is compliant with the AOP Alliance interface for around advice using method interception. MethodInterceptors implementing around advice should implement the following interface:

```java
public interface MethodInterceptor extends Interceptor {

	Object invoke(MethodInvocation invocation) throws Throwable;
}
```

The `MethodInvocation` argument to the `invoke()` method exposes the method being invoked; the target join point; the AOP proxy; and the arguments to the method. The `invoke()` method should return the invocation’s result: the return value of the join point.

A simple `MethodInterceptor` implementation looks as follows:

```java
public class DebugInterceptor implements MethodInterceptor {

	public Object invoke(MethodInvocation invocation) throws Throwable {
		System.out.println("Before: invocation=[" + invocation + "]");
		Object rval = invocation.proceed();
		System.out.println("Invocation returned");
		return rval;
	}
}
```

Note the call to the MethodInvocation’s `proceed()` method. This proceeds down the interceptor chain towards the join point. Most interceptors will invoke this method, and return its return value. However, a MethodInterceptor, like any around advice, can return a different value or throw an exception rather than invoke the proceed method. However, you don’t want to do this without good reason!

| ![[Note]](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| ---------------------------------------- |
| MethodInterceptors offer interoperability with other AOP Alliance-compliant AOP implementations. The other advice types discussed in the remainder of this section implement common AOP concepts, but in a Spring-specific way. While there is an advantage in using the most specific advice type, stick with MethodInterceptor around advice if you are likely to want to run the aspect in another AOP framework. Note that pointcuts are not currently interoperable between frameworks, and the AOP Alliance does not currently define pointcut interfaces. |

#### Before advice

A simpler advice type is a *before advice*. This does not need a `MethodInvocation` object, since it will only be called before entering the method.

The main advantage of a before advice is that there is no need to invoke the `proceed()` method, and therefore no possibility of inadvertently failing to proceed down the interceptor chain.

The `MethodBeforeAdvice` interface is shown below. (Spring’s API design would allow for field before advice, although the usual objects apply to field interception and it’s unlikely that Spring will ever implement it).

```java
public interface MethodBeforeAdvice extends BeforeAdvice {

	void before(Method m, Object[] args, Object target) throws Throwable;
}
```

Note the return type is `void`. Before advice can insert custom behavior before the join point executes, but cannot change the return value. If a before advice throws an exception, this will abort further execution of the interceptor chain. The exception will propagate back up the interceptor chain. If it is unchecked, or on the signature of the invoked method, it will be passed directly to the client; otherwise it will be wrapped in an unchecked exception by the AOP proxy.

An example of a before advice in Spring, which counts all method invocations:

```java
public class CountingBeforeAdvice implements MethodBeforeAdvice {

	private int count;

	public void before(Method m, Object[] args, Object target) throws Throwable {
		++count;
	}

	public int getCount() {
		return count;
	}
}
```

| ![[Tip]](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/images/tip.png.pagespeed.ce.w22Wv-tZ37.png) |
| ---------------------------------------- |
| Before advice can be used with any pointcut. |

#### Throws advice

*Throws advice* is invoked after the return of the join point if the join point threw an exception. Spring offers typed throws advice. Note that this means that the`org.springframework.aop.ThrowsAdvice` interface does not contain any methods: It is a tag interface identifying that the given object implements one or more typed throws advice methods. These should be in the form of:

```java
afterThrowing([Method, args, target], subclassOfThrowable)
```

Only the last argument is required. The method signatures may have either one or four arguments, depending on whether the advice method is interested in the method and arguments. The following classes are examples of throws advice.

The advice below is invoked if a `RemoteException` is thrown (including subclasses):

```java
public class RemoteThrowsAdvice implements ThrowsAdvice {

	public void afterThrowing(RemoteException ex) throws Throwable {
		// Do something with remote exception
	}
}
```

The following advice is invoked if a `ServletException` is thrown. Unlike the above advice, it declares 4 arguments, so that it has access to the invoked method, method arguments and target object:

```java
public class ServletThrowsAdviceWithArguments implements ThrowsAdvice {

	public void afterThrowing(Method m, Object[] args, Object target, ServletException ex) {
		// Do something with all arguments
	}
}
```

The final example illustrates how these two methods could be used in a single class, which handles both `RemoteException` and `ServletException`. Any number of throws advice methods can be combined in a single class.

```java
public static class CombinedThrowsAdvice implements ThrowsAdvice {

	public void afterThrowing(RemoteException ex) throws Throwable {
		// Do something with remote exception
	}

	public void afterThrowing(Method m, Object[] args, Object target, ServletException ex) {
		// Do something with all arguments
	}
}
```

| ![[Note]](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| ---------------------------------------- |
| If a throws-advice method throws an exception itself, it will override the original exception (i.e. change the exception thrown to the user). The overriding exception will typically be a RuntimeException; this is compatible with any method signature. However, if a throws-advice method throws a checked exception, it will have to match the declared exceptions of the target method and is hence to some degree coupled to specific target method signatures. *Do not throw an undeclared checked exception that is incompatible with the target method’s signature!* |

| ![[Tip]](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/images/tip.png.pagespeed.ce.w22Wv-tZ37.png) |
| ---------------------------------------- |
| Throws advice can be used with any pointcut. |

#### After Returning advice

An after returning advice in Spring must implement the *org.springframework.aop.AfterReturningAdvice* interface, shown below:

```java
public interface AfterReturningAdvice extends Advice {

	void afterReturning(Object returnValue, Method m, Object[] args, Object target)
			throws Throwable;
}
```

An after returning advice has access to the return value (which it cannot modify), invoked method, methods arguments and target.

The following after returning advice counts all successful method invocations that have not thrown exceptions:

```java
public class CountingAfterReturningAdvice implements AfterReturningAdvice {

	private int count;

	public void afterReturning(Object returnValue, Method m, Object[] args, Object target)
			throws Throwable {
		++count;
	}

	public int getCount() {
		return count;
	}
}
```

This advice doesn’t change the execution path. If it throws an exception, this will be thrown up the interceptor chain instead of the return value.

| ![[Tip]](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/images/tip.png.pagespeed.ce.w22Wv-tZ37.png) |
| ---------------------------------------- |
| After returning advice can be used with any pointcut. |

#### Introduction advice

Spring treats introduction advice as a special kind of interception advice.

Introduction requires an `IntroductionAdvisor`, and an `IntroductionInterceptor`, implementing the following interface:

```java
public interface IntroductionInterceptor extends MethodInterceptor {

	boolean implementsInterface(Class intf);
}
```

The `invoke()` method inherited from the AOP Alliance `MethodInterceptor` interface must implement the introduction: that is, if the invoked method is on an introduced interface, the introduction interceptor is responsible for handling the method call - it cannot invoke `proceed()`.

Introduction advice cannot be used with any pointcut, as it applies only at class, rather than method, level. You can only use introduction advice with the`IntroductionAdvisor`, which has the following methods:

```java
public interface IntroductionAdvisor extends Advisor, IntroductionInfo {

	ClassFilter getClassFilter();

	void validateInterfaces() throws IllegalArgumentException;
}

public interface IntroductionInfo {

	Class[] getInterfaces();
}
```

There is no `MethodMatcher`, and hence no `Pointcut`, associated with introduction advice. Only class filtering is logical.

The `getInterfaces()` method returns the interfaces introduced by this advisor.

The `validateInterfaces()` method is used internally to see whether or not the introduced interfaces can be implemented by the configured `IntroductionInterceptor`.

Let’s look at a simple example from the Spring test suite. Let’s suppose we want to introduce the following interface to one or more objects:

```java
public interface Lockable {
	void lock();
	void unlock();
	boolean locked();
}
```

This illustrates a *mixin*. We want to be able to cast advised objects to Lockable, whatever their type, and call lock and unlock methods. If we call the lock() method, we want all setter methods to throw a `LockedException`. Thus we can add an aspect that provides the ability to make objects immutable, without them having any knowledge of it: a good example of AOP.

Firstly, we’ll need an `IntroductionInterceptor` that does the heavy lifting. In this case, we extend the `org.springframework.aop.support.DelegatingIntroductionInterceptor` convenience class. We could implement IntroductionInterceptor directly, but using`DelegatingIntroductionInterceptor` is best for most cases.

The `DelegatingIntroductionInterceptor` is designed to delegate an introduction to an actual implementation of the introduced interface(s), concealing the use of interception to do so. The delegate can be set to any object using a constructor argument; the default delegate (when the no-arg constructor is used) is this. Thus in the example below, the delegate is the `LockMixin` subclass of `DelegatingIntroductionInterceptor`. Given a delegate (by default itself), a `DelegatingIntroductionInterceptor` instance looks for all interfaces implemented by the delegate (other than IntroductionInterceptor), and will support introductions against any of them. It’s possible for subclasses such as `LockMixin` to call the `suppressInterface(Class intf)` method to suppress interfaces that should not be exposed. However, no matter how many interfaces an `IntroductionInterceptor` is prepared to support, the `IntroductionAdvisor` used will control which interfaces are actually exposed. An introduced interface will conceal any implementation of the same interface by the target.

Thus `LockMixin` extends `DelegatingIntroductionInterceptor` and implements `Lockable` itself. The superclass automatically picks up that Lockable can be supported for introduction, so we don’t need to specify that. We could introduce any number of interfaces in this way.

Note the use of the `locked` instance variable. This effectively adds additional state to that held in the target object.

```java
public class LockMixin extends DelegatingIntroductionInterceptor implements Lockable {

	private boolean locked;

	public void lock() {
		this.locked = true;
	}

	public void unlock() {
		this.locked = false;
	}

	public boolean locked() {
		return this.locked;
	}

	public Object invoke(MethodInvocation invocation) throws Throwable {
		if (locked() && invocation.getMethod().getName().indexOf("set") == 0) {
			throw new LockedException();
		}
		return super.invoke(invocation);
	}

}
```

Often it isn’t necessary to override the `invoke()` method: the `DelegatingIntroductionInterceptor` implementation - which calls the delegate method if the method is introduced, otherwise proceeds towards the join point - is usually sufficient. In the present case, we need to add a check: no setter method can be invoked if in locked mode.

The introduction advisor required is simple. All it needs to do is hold a distinct `LockMixin` instance, and specify the introduced interfaces - in this case, just `Lockable`. A more complex example might take a reference to the introduction interceptor (which would be defined as a prototype): in this case, there’s no configuration relevant for a `LockMixin`, so we simply create it using `new`.

```java
public class LockMixinAdvisor extends DefaultIntroductionAdvisor {

	public LockMixinAdvisor() {
		super(new LockMixin(), Lockable.class);
	}
}
```

We can apply this advisor very simply: it requires no configuration. (However, it *is* necessary: It’s impossible to use an `IntroductionInterceptor` without an*IntroductionAdvisor*.) As usual with introductions, the advisor must be per-instance, as it is stateful. We need a different instance of `LockMixinAdvisor`, and hence`LockMixin`, for each advised object. The advisor comprises part of the advised object’s state.

We can apply this advisor programmatically, using the `Advised.addAdvisor()` method, or (the recommended way) in XML configuration, like any other advisor. All proxy creation choices discussed below, including "auto proxy creators," correctly handle introductions and stateful mixins.