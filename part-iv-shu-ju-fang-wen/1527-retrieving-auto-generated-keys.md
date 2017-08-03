### 15.2.7**获取自增Key**

`update()`方法支持获取数据库自增Key。这个支持已成为JDBC3.0标准之一、更多细节详见13.6章。这个方法使用`PreparedStatementCreator`作为其第一个入参，该类可以指定所需的insert语句。另外一个参数是`KeyHolder`，包含了更新操作成功之后产生的自增Key。这不是标准的创建`PreparedStatement `的方式。下面的例子可以在Oracle上面运行，但在其他平台上可能就不行了。

```java
final String INSERT_SQL = "insert into my_test (name) values(?)";
final String name = "Rob";

KeyHolder keyHolder = new GeneratedKeyHolder();
jdbcTemplate.update(
    new PreparedStatementCreator() {
        public PreparedStatement createPreparedStatement(Connection connection) throws SQLException {
            PreparedStatement ps = connection.prepareStatement(INSERT_SQL, new String[] {"id"});
            ps.setString(1, name);
            return ps;
        }
    },
    keyHolder);

// keyHolder.getKey() now contains the generated key
```



