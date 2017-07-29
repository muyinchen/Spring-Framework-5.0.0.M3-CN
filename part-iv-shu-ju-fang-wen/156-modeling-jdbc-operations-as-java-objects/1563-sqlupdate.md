### 15.6.3SqlUpdate

The`SqlUpdate`class encapsulates an SQL update. Like a query, an update object is reusable, and like all`RdbmsOperation`classes, an update can have parameters and is defined in SQL. This class provides a number of`update(..)`methods analogous to the`execute(..)`methods of query objects. The`SQLUpdate`class is concrete. It can be subclassed, for example, to add a custom update method, as in the following snippet where it’s simply called`execute`. However, you don’t have to subclass the`SqlUpdate`class since it can easily be parameterized by setting SQL and declaring parameters.

```java
import java.sql.Types;

import javax.sql.DataSource;

import org.springframework.jdbc.core.SqlParameter;
import org.springframework.jdbc.object.SqlUpdate;

public class UpdateCreditRating extends SqlUpdate {

	public UpdateCreditRating(DataSource ds) {
		setDataSource(ds);
		setSql("update customer set credit_rating = ? where id = ?");
		declareParameter(new SqlParameter("creditRating", Types.NUMERIC));
		declareParameter(new SqlParameter("id", Types.NUMERIC));
		compile();
	}

	/**
	 * @param id for the Customer to be updated
	 * @param rating the new value for credit rating
	 * @return number of rows updated
	 */
	public int execute(int id, int rating) {
		return update(rating, id);
	}
}
```



