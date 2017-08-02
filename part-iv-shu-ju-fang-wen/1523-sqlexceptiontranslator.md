### 15.2.3SQLExceptionTranslator

`SQLExceptionTranslator`接口用于在`SQLExceptions`和spring自己的`org.springframework.dao.DataAccessException`之间做转换，要处理批量更新或者从文件中这是为了屏蔽底层的数据访问策略。其实现可以是比较通用的（例如，使用JDBC的SQLState编码），或者是更精确专有的（例如，使用Oracle的错误类型编码）

`SQLExceptionTranslator `接口的默认实现是`SQLErrorCodeSQLExceptionTranslator`，该实现使用的是指定数据库厂商的错误编码，因为要比`SQLState`的实现更加精确。错误码转换过程基于JavaBean类型的`SQLErrorCodes`。这个类通过`SQLErrorCodesFactory`创建和返回，`SQLErrorCodesFactory`是一个基于`sql-error-codes.xml`配置内容来创建`SQLErrorCodes`的工厂类。该配置中的数据库厂商代码基于`Database MetaData`信息中返回的数据库产品名`DatabaseProductName`，最终使用的就是你正在使用的实际数据库中错误码。

`SQLErrorCodeSQLExceptionTranslator`按以下的顺序来匹配规则：

| ![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| :--- |
| 备注：`SQLErrorCodesFactory`是用于定义错误码和自定义异常转换的缺省工厂类。错误码参照classpath下配置的`sql-error-codes.xml`文件内容，相匹配的SQLErrorCodes实例基于正在使用的底层数据库的元数据名称。 |

* 是否存在自定义转换的子类。通常直接使用`SQLErrorCodeSQLExceptionTranslator`就可以了，因此此规则一般不会生效。只有你真正自己实现了一个子类才会生效。

* 是否存在`SQLExceptionTranslator`接口的自定义实现，通过`SQLErrorCodes`类的`customSqlExceptionTranslator`属性指定

* `SQLErrorCodes`的`customTranslations`属性数组、类型为`CustomSQLErrorCodesTranslation`类实例列表、能否被匹配到

* 错误码被匹配到

* 使用兜底的转换器。`SQLExceptionSubclassTranslator`是缺省的兜底转换器。如果此转换器也不存在的话只能使用`SQLStateSQLExceptionTranslator`

你可以继承`SQLErrorCodeSQLExceptionTranslator`：

```java
public class CustomSQLErrorCodesTranslator extends SQLErrorCodeSQLExceptionTranslator {

    protected DataAccessException customTranslate(String task, String sql, SQLException sqlex) {
        if (sqlex.getErrorCode() == -12345) {
            return new DeadlockLoserDataAccessException(task, sqlex);
        }
        return null;
    }
}
```

这个例子中，特定的错误码`-12345`被识别后单独转换，而其他的错误码则通过默认的转换器实现来处理。在使用自定义转换器时，有必要通过`setExceptionTranslator`方法传入`JdbcTemplate `，并且使用`JdbcTemplate`来做所有的数据访问处理。下面是一个如何使用自定义转换器的例子

```java
private JdbcTemplate jdbcTemplate;

public void setDataSource(DataSource dataSource) {

    // create a JdbcTemplate and set data source
    this.jdbcTemplate = new JdbcTemplate();
    this.jdbcTemplate.setDataSource(dataSource);

    // create a custom translator and set the DataSource for the default translation lookup
    CustomSQLErrorCodesTranslator tr = new CustomSQLErrorCodesTranslator();
    tr.setDataSource(dataSource);
    this.jdbcTemplate.setExceptionTranslator(tr);

}

public void updateShippingCharge(long orderId, long pct) {
    // use the prepared JdbcTemplate for this update
    this.jdbcTemplate.update("update orders" +
        " set shipping_charge = shipping_charge * ? / 100" +
        " where id = ?", pct, orderId);
}
```

自定义转换器需要传入dataSource对象为了能够获取`sql-error-codes.xml`定义的错误码

