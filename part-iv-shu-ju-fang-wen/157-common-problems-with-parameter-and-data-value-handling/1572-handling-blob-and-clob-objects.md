### 15.7.2Handling BLOB and CLOB objects

You can store images, other binary data, and large chunks of text in the database. These large objects are called BLOBs \(Binary Large OBject\) for binary data and CLOBs \(Character Large OBject\) for character data. In Spring you can handle these large objects by using the`JdbcTemplate`directly and also when using the higher abstractions provided by RDBMS Objects and the`SimpleJdbc`classes. All of these approaches use an implementation of the`LobHandler`interface for the actual management of the LOB \(Large OBject\) data. The`LobHandler`provides access to a`LobCreator`class, through the`getLobCreator`method, used for creating new LOB objects to be inserted.

The`LobCreator/LobHandler`provides the following support for LOB input and output:

* BLOB

  * `byte[]` — `getBlobAsBytes`and`setBlobAsBytes`

  * `InputStream` — `getBlobAsBinaryStream`and`setBlobAsBinaryStream`

* CLOB

  * `String` — `getClobAsString`and`setClobAsString`

  * `InputStream` — `getClobAsAsciiStream`and`setClobAsAsciiStream`

  * `Reader` — `getClobAsCharacterStream`and`setClobAsCharacterStream`

The next example shows how to create and insert a BLOB. Later you will see how to read it back from the database.

This example uses a`JdbcTemplate`and an implementation of the`AbstractLobCreatingPreparedStatementCallback`. It implements one method,`setValues`. This method provides a`LobCreator`that you use to set the values for the LOB columns in your SQL insert statement.

For this example we assume that there is a variable,`lobHandler`, that already is set to an instance of a`DefaultLobHandler`. You typically set this value through dependency injection.

```
final File blobIn = new File("spring2004.jpg");
final InputStream blobIs = new FileInputStream(blobIn);
final File clobIn = new File("large.txt");
final InputStream clobIs = new FileInputStream(clobIn);
final InputStreamReader clobReader = new InputStreamReader(clobIs);
jdbcTemplate.execute(
	"INSERT INTO lob_table (id, a_clob, a_blob) VALUES (?, ?, ?)",
	new AbstractLobCreatingPreparedStatementCallback(lobHandler) { //1
		protected void setValues(PreparedStatement ps, LobCreator lobCreator) throws SQLException {
			ps.setLong(1, 1L);
			lobCreator.setClobAsCharacterStream(ps, 2, clobReader, (int)clobIn.length()); //2
			lobCreator.setBlobAsBinaryStream(ps, 3, blobIs, (int)blobIn.length()); //3
		}
	}
);
blobIs.close();
clobReader.close();
```

1. Pass in the`lobHandler`that in this example is a plain`DefaultLobHandler`.

2. Using the method `setClobAsCharacterStream`, pass in the contents of the CLOB

3. Using the method`setBlobAsBinaryStream`, pass in the contents of the BLOB.

| ![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| :--- |
| If you invoke the`setBlobAsBinaryStream`,`setClobAsAsciiStream`, or`setClobAsCharacterStream`method on the`LobCreator`returned from`DefaultLobHandler.getLobCreator()`, you can optionally specify a negative value for the`contentLength`argument. If the specified content length is negative, the`DefaultLobHandler`will use the JDBC 4.0 variants of the set-stream methods without a length parameter; otherwise, it will pass the specified length on to the driver.Consult the documentation for the JDBC driver in use to verify support for streaming a LOB without providing the content length. |

Now it’s time to read the LOB data from the database. Again, you use a`JdbcTemplate`with the same instance variable`lobHandler`and a reference to a`DefaultLobHandler`.

```java
List<Map<String, Object>> l = jdbcTemplate.query("select id, a_clob, a_blob from lob_table",
	new RowMapper<Map<String, Object>>() {
		public Map<String, Object> mapRow(ResultSet rs, int i) throws SQLException {
			Map<String, Object> results = new HashMap<String, Object>();
			String clobText = lobHandler.getClobAsString(rs, "a_clob");   //1
results.put("CLOB", clobText); byte[] blobBytes = lobHandler.getBlobAsBytes(rs, "a_blob");  //2
results.put("BLOB", blobBytes); return results; } });
```

1. Using the method `getClobAsString`, retrieve the contents of the CLOB

2. Using the method`getBlobAsBytes`, retrieve the contents of the BLOB.



