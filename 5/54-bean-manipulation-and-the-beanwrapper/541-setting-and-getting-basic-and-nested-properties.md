### 5.4.1 设置并获取基本和嵌套属性

使用`setPropertyValue(s)`和`getPropertyValue(s)`可以设置并获取属性，两者都带有几个重载方法。在Spring自带的java文档中对它们有更详细的描述。重要的是要知道对象属性指示的几个约定。几个例子：

**表 5.1. 属性示例**

| 表达式                    | 说明                                       |
| ---------------------- | ---------------------------------------- |
| `name`                 | 表示属性`name`与方法`getName()`或`isName()`和`setName()`相对应 |
| `account.name`         | 表示属性`account`的嵌套属性`name`与方法`getAccount().setName()`或`getAccount().getName()`相对应 |
| `account[2]`           | 表示索引属性`account`的第三个元素。索引属性可以是`array`、`list`或其他自然排序的集合 |
| `account[COMPANYNAME]` | 表示映射属性`account`被键*COMPANYNAME*索引到的映射项的值  |

下面你会发现一些使用`BeanWrapper`来获取和设置属性的例子。

*(如果你不打算直接使用`BeanWrapper`，那么下一部分对你来说并不重要。如果你仅使用`DataBinder`和`BeanFactory`以及它们开箱即用的实现，你应该跳到关于`PropertyEditor`部分的开头)。*

考虑下面两个类：

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

以下的代码片段展示了如何检索和操纵实例化的`Companies`和`Employees`的某些属性：

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