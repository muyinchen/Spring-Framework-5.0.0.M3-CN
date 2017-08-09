### 15.6.3SqlUpdate

`SqlUpdate`封装了SQL的更新操作。和查询一样，更新对象是可以被重用的，就像所有的`RdbmsOperation`类，更新操作能够传入参数并且在SQL定义。类似于`SqlQuery`诸多`execute(..)`方法，这个类提供了一系列`update(..)`方法。`SQLUpdate`类不是抽象类，它可以被继承，比如，实现自定义的更新方法。但是你并不需要继承`SqlUpdate`类来达到这个目的，你可以更简单的在SQL中设置自定义参数来实现。

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



