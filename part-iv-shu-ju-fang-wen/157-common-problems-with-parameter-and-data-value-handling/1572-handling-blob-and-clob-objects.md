### 15.7.2**处理BLOB和CLOB对象**

你可以存储图片，其他类型的二进制数据，和数据库里面的大块文本。这些大的对象叫做BLOBS（全称：Binary Large OBject；用于二进制数据）和CLOBS（全称：Character Large OBject；用于字符数据）。在Spring中你可以使用`JdbcTemplate`直接处理这些大对象，并且也可以使用RDBMS对象提供的上层抽象类，或者使用`SimpleJdbc`类。所有这些方法使用`LobHandler`接口的实现类来处理LOB数据的管理（全称：Large Object）。`LobHandler`通过`getLobCreator`方法提供了对`LobCreator `类的访问，用于创建新的LOB插入对象。

`LobCreator/LobHandler`提供了LOB输入和输出的支持：

* BLOB

  * `byte[]` — `getBlobAsBytes`和`setBlobAsBytes`

  * `InputStream` — `getBlobAsBinaryStream`和`setBlobAsBinaryStream`

* CLOB

  * `String` — `getClobAsString`和`setClobAsString`

  * `InputStream` — `getClobAsAsciiStream`和`setClobAsAsciiStream`

  * `Reader` — `getClobAsCharacterStream`和`setClobAsCharacterStream`

下面的例子展示了如何创建和插入一个BLOB。后面的例子我们将举例如何从数据库中将BLOB数据读取出来

这个例子使用了`JdbcTemplate`和`AbstractLobCreatingPreparedStatementCallback`的实现类。它主要实现了一个方法，`setValues`.这个方法提供了用于在你的SQL插入语句中设置LOB列的`LobCreator`。

针对这个例子我们假定有一个变量`lobHandler`，已经设置了`DefaultLobHandler`的一个实例。通常你可以使用依赖注入来设置这个值。

```java
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

1. 这里例子中传入的`lobHandler `使用了默认实现类`DefaultLobHandler`

2. 使用`setClobAsCharacterStream`传入CLOB的内容

3. 使用`setBlobAsBinaryStream`传入BLOB的内容

| ![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| :--- |
| 备注：如果你调用从`DefaultLobHandler.getLobCreator()`返回的`LobCreator`的`setBlobAsBinaryStream`, `setClobAsAsciiStream`, 或者`setClobAsCharacterStream`方法，其中`contentLength`参数允许传入一个负值。如果指定的内容长度是负值，`DefaultLobHandler`会使用JDBC4.0不带长度参数的set-stream方法，或者直接传入驱动指定的长度；JDBC驱动对未指定长度的LOB流的支持请参见相关文档. |

下面是从数据库读取LOB数据的例子。我们这里再次使用`JdbcTempate`并使用相同的`DefaultLobHandler`实例。

```java
List<Map<String, Object>> l = jdbcTemplate.query("select id, a_clob, a_blob from lob_table",
    new RowMapper<Map<String, Object>>() {
        public Map<String, Object> mapRow(ResultSet rs, int i) throws SQLException {
            Map<String, Object> results = new HashMap<String, Object>();
            String clobText = lobHandler.getClobAsString(rs, "a_clob");   //1
results.put("CLOB", clobText); byte[] blobBytes = lobHandler.getBlobAsBytes(rs, "a_blob");  //2
results.put("BLOB", blobBytes); return results; } });
```

1. 使用`getClobAsString`获取CLOB的内容

2. 使用`getBlobAsBytes`获取BLOC的内容



