## 5.8 Spring Validation

Spring 3 introduces several enhancements to its validation support. First, the JSR-303 Bean Validation API is now fully supported. Second, when used programmatically, Spring’s DataBinder can now validate objects as well as bind to them. Third, Spring MVC now has support for declaratively validating `@Controller` inputs.

### 5.8.1 Overview of the JSR-303 Bean Validation API

JSR-303 standardizes validation constraint declaration and metadata for the Java platform. Using this API, you annotate domain model properties with declarative validation constraints and the runtime enforces them. There are a number of built-in constraints you can take advantage of. You may also define your own custom constraints.

To illustrate, consider a simple PersonForm model with two properties:

```java
public class PersonForm {
	private String name;
	private int age;
}
```

JSR-303 allows you to define declarative validation constraints against such properties:

```java
public class PersonForm {

	@NotNull
	@Size(max=64)
	private String name;

	@Min(0)
	private int age;

}
```

When an instance of this class is validated by a JSR-303 Validator, these constraints will be enforced.

For general information on JSR-303/JSR-349, see the [Bean Validation website](http://beanvalidation.org/). For information on the specific capabilities of the default reference implementation, see the [Hibernate Validator](https://www.hibernate.org/412.html) documentation. To learn how to setup a Bean Validation provider as a Spring bean, keep reading.

### 5.8.2 Configuring a Bean Validation Provider

Spring provides full support for the Bean Validation API. This includes convenient support for bootstrapping a JSR-303/JSR-349 Bean Validation provider as a Spring bean. This allows for a `javax.validation.ValidatorFactory` or `javax.validation.Validator` to be injected wherever validation is needed in your application.

Use the `LocalValidatorFactoryBean` to configure a default Validator as a Spring bean:

```xml
<bean id="validator"
	class="org.springframework.validation.beanvalidation.LocalValidatorFactoryBean"/>
```

The basic configuration above will trigger Bean Validation to initialize using its default bootstrap mechanism. A JSR-303/JSR-349 provider, such as Hibernate Validator, is expected to be present in the classpath and will be detected automatically.

#### Injecting a Validator

`LocalValidatorFactoryBean` implements both `javax.validation.ValidatorFactory` and `javax.validation.Validator`, as well as Spring’s`org.springframework.validation.Validator`. You may inject a reference to either of these interfaces into beans that need to invoke validation logic.

Inject a reference to `javax.validation.Validator` if you prefer to work with the Bean Validation API directly:

```java
import javax.validation.Validator;

@Service
public class MyService {

	@Autowired
	private Validator validator;
```

Inject a reference to `org.springframework.validation.Validator` if your bean requires the Spring Validation API:

```java
import org.springframework.validation.Validator;

@Service
public class MyService {

	@Autowired
	private Validator validator;

}
```

#### Configuring Custom Constraints

Each Bean Validation constraint consists of two parts. First, a `@Constraint` annotation that declares the constraint and its configurable properties. Second, an implementation of the `javax.validation.ConstraintValidator` interface that implements the constraint’s behavior. To associate a declaration with an implementation, each `@Constraint` annotation references a corresponding ValidationConstraint implementation class. At runtime, a `ConstraintValidatorFactory`instantiates the referenced implementation when the constraint annotation is encountered in your domain model.

By default, the `LocalValidatorFactoryBean` configures a `SpringConstraintValidatorFactory` that uses Spring to create ConstraintValidator instances. This allows your custom ConstraintValidators to benefit from dependency injection like any other Spring bean.

Shown below is an example of a custom `@Constraint` declaration, followed by an associated `ConstraintValidator` implementation that uses Spring for dependency injection:

```java
@Target({ElementType.METHOD, ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy=MyConstraintValidator.class)
public @interface MyConstraint {
}
```

```java
import javax.validation.ConstraintValidator;

public class MyConstraintValidator implements ConstraintValidator {

	@Autowired;
	private Foo aDependency;

	...
}
```

As you can see, a ConstraintValidator implementation may have its dependencies @Autowired like any other Spring bean.

#### Spring-driven Method Validation

The method validation feature supported by Bean Validation 1.1, and as a custom extension also by Hibernate Validator 4.3, can be integrated into a Spring context through a `MethodValidationPostProcessor` bean definition:

```xml
<bean class="org.springframework.validation.beanvalidation.MethodValidationPostProcessor"/>
```

In order to be eligible for Spring-driven method validation, all target classes need to be annotated with Spring’s `@Validated` annotation, optionally declaring the validation groups to use. Check out the `MethodValidationPostProcessor` javadocs for setup details with Hibernate Validator and Bean Validation 1.1 providers.

#### Additional Configuration Options

The default `LocalValidatorFactoryBean` configuration should prove sufficient for most cases. There are a number of configuration options for various Bean Validation constructs, from message interpolation to traversal resolution. See the `LocalValidatorFactoryBean` javadocs for more information on these options.

### 5.8.3 Configuring a DataBinder

Since Spring 3, a DataBinder instance can be configured with a Validator. Once configured, the Validator may be invoked by calling `binder.validate()`. Any validation Errors are automatically added to the binder’s BindingResult.

When working with the DataBinder programmatically, this can be used to invoke validation logic after binding to a target object:

```java
Foo target = new Foo();
DataBinder binder = new DataBinder(target);
binder.setValidator(new FooValidator());

// bind to the target object
binder.bind(propertyValues);

// validate the target object
binder.validate();

// get BindingResult that includes any validation errors
BindingResult results = binder.getBindingResult();
```

A DataBinder can also be configured with multiple `Validator` instances via `dataBinder.addValidators` and `dataBinder.replaceValidators`. This is useful when combining globally configured Bean Validation with a Spring `Validator` configured locally on a DataBinder instance. See [???](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#).

### 5.8.4 Spring MVC 3 Validation

See [Section 18.16.4, “Validation”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#mvc-config-validation) in the Spring MVC chapter.