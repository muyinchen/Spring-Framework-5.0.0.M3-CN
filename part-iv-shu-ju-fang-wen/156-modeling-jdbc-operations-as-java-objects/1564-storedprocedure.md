### 15.6.4StoredProcedure

The`StoredProcedure`class is a superclass for object abstractions of RDBMS stored procedures. This class is`abstract`, and its various`execute(..)`methods have`protected`access, preventing use other than through a subclass that offers tighter typing.

The inherited`sql`property will be the name of the stored procedure in the RDBMS.

To define a parameter for the`StoredProcedure`class, you use an`SqlParameter`or one of its subclasses. You must specify the parameter name and SQL type in the constructor like in the following code snippet. The SQL type is specified using the`java.sql.Types`constants.

```java
new SqlParameter("in_id", Types.NUMERIC),
	new SqlOutParameter("out_first_name", Types.VARCHAR),
```

The first line with the`SqlParameter`declares an IN parameter. IN parameters can be used for both stored procedure calls and for queries using the`SqlQuery`and its subclasses covered in the following section.

The second line with the`SqlOutParameter`declares an`out`parameter to be used in the stored procedure call. There is also an`SqlInOutParameter`for`InOut`parameters, parameters that provide an`in`value to the procedure and that also return a value.

For`in`parameters, in addition to the name and the SQL type, you can specify a scale for numeric data or a type name for custom database types. For`out`parameters you can provide a`RowMapper`to handle mapping of rows returned from a REF cursor. Another option is to specify an`SqlReturnType`that enables you to define customized handling of the return values.

Here is an example of a simple DAO that uses a`StoredProcedure`to call a function,`sysdate()`,which comes with any Oracle database. To use the stored procedure functionality you have to create a class that extends`StoredProcedure`. In this example, the`StoredProcedure`class is an inner class, but if you need to reuse the`StoredProcedure`you declare it as a top-level class. This example has no input parameters, but an output parameter is declared as a date type using the class`SqlOutParameter`. The`execute()`method executes the procedure and extracts the returned date from the results`Map`. The results`Map`has an entry for each declared output parameter, in this case only one, using the parameter name as the key.

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

The following example of a`StoredProcedure`has two output parameters \(in this case, Oracle REF cursors\).

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

Notice how the overloaded variants of the`declareParameter(..)`method that have been used in the`TitlesAndGenresStoredProcedure`constructor are passed`RowMapper`implementation instances; this is a very convenient and powerful way to reuse existing functionality. The code for the two`RowMapper`implementations is provided below.

The`TitleMapper`class maps a`ResultSet`to a`Title`domain object for each row in the supplied`ResultSet`:

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

The`GenreMapper`class maps a`ResultSet`to a`Genre`domain object for each row in the supplied`ResultSet`.

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

To pass parameters to a stored procedure that has one or more input parameters in its definition in the RDBMS, you can code a strongly typed`execute(..)`method that would delegate to the superclass' untyped`execute(Map parameters)`method \(which has`protected`access\); for example:

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



