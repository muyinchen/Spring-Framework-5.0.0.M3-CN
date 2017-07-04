### 15.2.3SQLExceptionTranslator

`SQLExceptionTranslator`is an interface to be implemented by classes that can translate between`SQLExceptions`and Spring’s own`org.springframework.dao.DataAccessException`, which is agnostic in regard to data access strategy. Implementations can be generic \(for example, using SQLState codes for JDBC\) or proprietary \(for example, using Oracle error codes\) for greater precision.

`SQLErrorCodeSQLExceptionTranslator`is the implementation of`SQLExceptionTranslator`that is used by default. This implementation uses specific vendor codes. It is more precise than the`SQLState`implementation. The error code translations are based on codes held in a JavaBean type class called`SQLErrorCodes`. This class is created and populated by an`SQLErrorCodesFactory`which as the name suggests is a factory for creating`SQLErrorCodes`based on the contents of a configuration file named`sql-error-codes.xml`. This file is populated with vendor codes and based on the`DatabaseProductName`taken from the`DatabaseMetaData`. The codes for the actual database you are using are used.

The`SQLErrorCodeSQLExceptionTranslator`applies matching rules in the following sequence:

| ![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| :--- |
| The`SQLErrorCodesFactory`is used by default to define Error codes and custom exception translations. They are looked up in a file named`sql-error-codes.xml`from the classpath and the matching`SQLErrorCodes`instance is located based on the database name from the database metadata of the database in use. |

* Any custom translation implemented by a subclass. Normally the provided concrete`SQLErrorCodeSQLExceptionTranslator`is used so this rule does not apply. It only applies if you have actually provided a subclass implementation.

* Any custom implementation of the`SQLExceptionTranslator`interface that is provided as the`customSqlExceptionTranslator`property of the`SQLErrorCodes`class.

* The list of instances of the`CustomSQLErrorCodesTranslation`class, provided for the`customTranslations`property of the`SQLErrorCodes`class, are searched for a match.

* Error code matching is applied.

* Use the fallback translator.`SQLExceptionSubclassTranslator`is the default fallback translator. If this translation is not available then the next fallback translator is the`SQLStateSQLExceptionTranslator`.

You can extend`SQLErrorCodeSQLExceptionTranslator:`

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

In this example, the specific error code`-12345`is translated and other errors are left to be translated by the default translator implementation. To use this custom translator, it is necessary to pass it to the`JdbcTemplate`through the method`setExceptionTranslator`and to use this`JdbcTemplate`for all of the data access processing where this translator is needed. Here is an example of how this custom translator can be used:

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

The custom translator is passed a data source in order to look up the error codes in`sql-error-codes.xml`.

