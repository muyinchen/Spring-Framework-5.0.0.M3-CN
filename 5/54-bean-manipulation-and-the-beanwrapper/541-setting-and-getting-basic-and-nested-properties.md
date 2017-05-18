### 5.4.1 Setting and getting basic and nested properties

Setting and getting properties is done using the `setPropertyValue(s)` and `getPropertyValue(s)` methods that both come with a couple of overloaded variants. They’re all described in more detail in the javadocs Spring comes with. What’s important to know is that there are a couple of conventions for indicating properties of an object. A couple of examples:

**Table 5.1. Examples of properties**

| Expression             | Explanation                              |
| ---------------------- | ---------------------------------------- |
| `name`                 | Indicates the property `name` corresponding to the methods `getName()` or `isName()` and `setName(..)` |
| `account.name`         | Indicates the nested property `name` of the property `account` corresponding e.g. to the methods `getAccount().setName()` or `getAccount().getName()` |
| `account[2]`           | Indicates the *third* element of the indexed property `account`. Indexed properties can be of type `array`, `list` or other *naturally ordered* collection |
| `account[COMPANYNAME]` | Indicates the value of the map entry indexed by the key *COMPANYNAME* of the Map property `account` |

Below you’ll find some examples of working with the `BeanWrapper` to get and set properties.

*(This next section is not vitally important to you if you’re not planning to work with the BeanWrapper directly. If you’re just using the DataBinder and the BeanFactoryand their out-of-the-box implementation, you should skip ahead to the section about PropertyEditors.)*

Consider the following two classes:

```java
public class Company {

	private String name;
	private Employee managingDirector;

	public String getName() {
		return this.name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public Employee getManagingDirector() {
		return this.managingDirector;
	}

	public void setManagingDirector(Employee managingDirector) {
		this.managingDirector = managingDirector;
	}
}
```

```java
public class Employee {

	private String name;

	private float salary;

	public String getName() {
		return this.name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public float getSalary() {
		return salary;
	}

	public void setSalary(float salary) {
		this.salary = salary;
	}
}
```

The following code snippets show some examples of how to retrieve and manipulate some of the properties of instantiated `Companies` and `Employees`:

```java
BeanWrapper company = new BeanWrapperImpl(new Company());
// setting the company name..
company.setPropertyValue("name", "Some Company Inc.");
// ... can also be done like this:
PropertyValue value = new PropertyValue("name", "Some Company Inc.");
company.setPropertyValue(value);

// ok, let's create the director and tie it to the company:
BeanWrapper jim = new BeanWrapperImpl(new Employee());
jim.setPropertyValue("name", "Jim Stravinsky");
company.setPropertyValue("managingDirector", jim.getWrappedInstance());

// retrieving the salary of the managingDirector through the company
Float salary = (Float) company.getPropertyValue("managingDirector.salary");
```

