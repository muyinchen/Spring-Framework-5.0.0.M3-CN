### 15.6.1 SqlQuery

`SqlQuery`is a reusable, threadsafe class that encapsulates an SQL query. Subclasses must implement the`newRowMapper(..)`method to provide a`RowMapper`instance that can create one object per row obtained from iterating over the`ResultSet`that is created during the execution of the query. The`SqlQuery`class is rarely used directly because the`MappingSqlQuery`subclass provides a much more convenient implementation for mapping rows to Java classes. Other implementations that extend`SqlQuery`are`MappingSqlQueryWithParameters`and`UpdatableSqlQuery`.

