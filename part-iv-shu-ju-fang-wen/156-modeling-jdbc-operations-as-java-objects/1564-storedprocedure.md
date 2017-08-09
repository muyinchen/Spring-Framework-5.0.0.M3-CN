### 15.6.4StoredProcedure

`StoredProcedure`类是所有RDBMS存储过程的抽象类。该类提供了多种`execute(..)`方法，其访问类型都是`protected`的。而不是通过提供更紧密打字的子类。

继承的`sql`属性将是RDBMS中存储过程的名称。

为了定义一个`StoredProcedure`类，你需要使用`SqlParameter`或者它的一个子类。你必须像下面的代码例子那样在构造函数中指定参数名和SQL类型。SQL类型使用`java.sql.Types`常量定义。

```java
new SqlParameter("in_id", Types.NUMERIC),
    new SqlOutParameter("out_first_name", Types.VARCHAR),
```

`SqlParameter`的第一行定义了一个输入参数。输入参数可以同时被存储过程调用和使用`SqlQuery`的查询语句使用，他的子类会在下面的章节提到。

第二行`SqlOutParameter`参数定义了一个在存储过程调用中使用的输出参数。`SqlInOutParameter`还有一个`InOut`参数，该参数提供了一个输入值，同时也有返回值。

对应输入参数，除了名字和SQL类型，你还能指定返回区间数值类型和自定义数据库类型。对于输出参数你可以使用`RowMapper`来处理REF游标返回的行映射关系。另一个选择是指定`SqlReturnType`，能够让你定义自定义的返回值类型。

下面的程序演示了如何调用Oracle中的`sysdate()`函数。为了使用存储过程函数你需要创建一个`StoredProcedure`的子类。在这个例子中，`StoredProcedure`是一个内部类，但是如果你需要重用`StoredProcedure`你需要定义成一个顶级类。这个例子没有输入参数，但是使用`SqlOutParameter`类定义了一个时间类型的输出参数。`execute()`方法执行了存储过程，并且从结果集`Map`中获取返回的时间数据。结果集`Map`中包含每个输出参数对应的项,在这个例子中就只有一项，使用了参数名作为`key`.

```java
import java.sql.Types;
import java.util.Date;
import java.util.HashMap;
import java.util.Map;

import javax.sql.DataSource;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.SqlOutParameter;
import org.springframework.jdbc.object.StoredProcedure;

public class StoredProcedureDao {

    private GetSysdateProcedure getSysdate;

    @Autowired
    public void init(DataSource dataSource) {
        this.getSysdate = new GetSysdateProcedure(dataSource);
    }

    public Date getSysdate() {
        return getSysdate.execute();
    }

    private class GetSysdateProcedure extends StoredProcedure {

        private static final String SQL = "sysdate";

        public GetSysdateProcedure(DataSource dataSource) {
            setDataSource(dataSource);
            setFunction(true);
            setSql(SQL);
            declareParameter(new SqlOutParameter("date", Types.DATE));
            compile();
        }

        public Date execute() {
            // the 'sysdate' sproc has no input parameters, so an empty Map is supplied...
            Map<String, Object> results = execute(new HashMap<String, Object>());
            Date sysdate = (Date) results.get("date");
            return sysdate;
        }
    }

}
```

下面是一个包含两个输出参数的存储过程例子（在这种情况下为Oracle REF游标）。

```java
import oracle.jdbc.OracleTypes;
import org.springframework.jdbc.core.SqlOutParameter;
import org.springframework.jdbc.object.StoredProcedure;

import javax.sql.DataSource;
import java.util.HashMap;
import java.util.Map;

public class TitlesAndGenresStoredProcedure extends StoredProcedure {

    private static final String SPROC_NAME = "AllTitlesAndGenres";

    public TitlesAndGenresStoredProcedure(DataSource dataSource) {
        super(dataSource, SPROC_NAME);
        declareParameter(new SqlOutParameter("titles", OracleTypes.CURSOR, new TitleMapper()));
        declareParameter(new SqlOutParameter("genres", OracleTypes.CURSOR, new GenreMapper()));
        compile();
    }

    public Map<String, Object> execute() {
        // again, this sproc has no input parameters, so an empty Map is supplied
        return super.execute(new HashMap<String, Object>());
    }
}
```

值得注意的是`TitlesAndGenresStoredProcedure`构造函数中使用过的`declareParameter(..)`方法的重载变量是如何传递`RowMapper`实现实例的。这是一种非常方便有效的重用方式。两种`RowMapper`实现的代码如下：

`TitleMapper`类将`ResultSet`映射到提供的`ResultSet`中每行的一个`Title`域对象：

```java
import org.springframework.jdbc.core.RowMapper;

import java.sql.ResultSet;
import java.sql.SQLException;

import com.foo.domain.Title;

public final class TitleMapper implements RowMapper<Title> {

    public Title mapRow(ResultSet rs, int rowNum) throws SQLException {
        Title title = new Title();
        title.setId(rs.getLong("id"));
        title.setName(rs.getString("name"));
        return title;
    }
}
```

`GenreMapper`类将返回结果集的每一行映射成`Genre`类

```java
import org.springframework.jdbc.core.RowMapper;

import java.sql.ResultSet;
import java.sql.SQLException;

import com.foo.domain.Genre;

public final class GenreMapper implements RowMapper<Genre> {

    public Genre mapRow(ResultSet rs, int rowNum) throws SQLException {
        return new Genre(rs.getString("name"));
    }
}
```

为了将参数传递给RDBMS中定义的一个或多个输入参数给存储过程，你可以定义一个强类型的`execute(..)`方法，该方法将调用基类的protected `execute(Map parameters)`方法（具有受保护的访问权限）。例如：

```java
import oracle.jdbc.OracleTypes;
import org.springframework.jdbc.core.SqlOutParameter;
import org.springframework.jdbc.core.SqlParameter;
import org.springframework.jdbc.object.StoredProcedure;

import javax.sql.DataSource;

import java.sql.Types;
import java.util.Date;
import java.util.HashMap;
import java.util.Map;

public class TitlesAfterDateStoredProcedure extends StoredProcedure {

    private static final String SPROC_NAME = "TitlesAfterDate";
    private static final String CUTOFF_DATE_PARAM = "cutoffDate";

    public TitlesAfterDateStoredProcedure(DataSource dataSource) {
        super(dataSource, SPROC_NAME);
        declareParameter(new SqlParameter(CUTOFF_DATE_PARAM, Types.DATE);
        declareParameter(new SqlOutParameter("titles", OracleTypes.CURSOR, new TitleMapper()));
        compile();
    }

    public Map<String, Object> execute(Date cutoffDate) {
        Map<String, Object> inputs = new HashMap<String, Object>();
        inputs.put(CUTOFF_DATE_PARAM, cutoffDate);
        return super.execute(inputs);
    }
}
```



