### 15.7.4Handling complex types for stored procedure calls

When you call stored procedures you can sometimes use complex types specific to the database. To accommodate these types, Spring provides a`SqlReturnType`for handling them when they are returned from the stored procedure call and`SqlTypeValue`when they are passed in as a parameter to the stored procedure.

Here is an example of returning the value of an Oracle`STRUCT`object of the user declared type`ITEM_TYPE`. The`SqlReturnType`interface has a single method named`getTypeValue`that must be implemented. This interface is used as part of the declaration of an`SqlOutParameter`.

```java
final TestItem = new TestItem(123L, "A test item",
		new SimpleDateFormat("yyyy-M-d").parse("2010-12-31"));

declareParameter(new SqlOutParameter("item", OracleTypes.STRUCT, "ITEM_TYPE",
	new SqlReturnType() {
		public Object getTypeValue(CallableStatement cs, int colIndx, int sqlType, String typeName) throws SQLException {
			STRUCT struct = (STRUCT) cs.getObject(colIndx);
			Object[] attr = struct.getAttributes();
			TestItem item = new TestItem();
			item.setId(((Number) attr[0]).longValue());
			item.setDescription((String) attr[1]);
			item.setExpirationDate((java.util.Date) attr[2]);
			return item;
		}
	}));
```

You use the`SqlTypeValue`to pass in the value of a Java object like`TestItem`into a stored procedure. The`SqlTypeValue`interface has a single method named`createTypeValue`that you must implement. The active connection is passed in, and you can use it to create database-specific objects such as`StructDescriptor`s, as shown in the following example, or`ArrayDescriptor`s.

```
final TestItem = new TestItem(123L, "A test item",
		new SimpleDateFormat("yyyy-M-d").parse("2010-12-31"));

SqlTypeValue value = new AbstractSqlTypeValue() {
	protected Object createTypeValue(Connection conn, int sqlType, String typeName) throws SQLException {
		StructDescriptor itemDescriptor = new StructDescriptor(typeName, conn);
		Struct item = new STRUCT(itemDescriptor, conn,
		new Object[] {
			testItem.getId(),
			testItem.getDescription(),
			new java.sql.Date(testItem.getExpirationDate().getTime())
		});
		return item;
	}
};
```

This`SqlTypeValue`can now be added to the Map containing the input parameters for the execute call of the stored procedure.

Another use for the`SqlTypeValue`is passing in an array of values to an Oracle stored procedure. Oracle has its own internal`ARRAY`class that must be used in this case, and you can use the`SqlTypeValue`to create an instance of the Oracle`ARRAY`and populate it with values from the Java`ARRAY`.

```java
final Long[] ids = new Long[] {1L, 2L};

SqlTypeValue value = new AbstractSqlTypeValue() {
	protected Object createTypeValue(Connection conn, int sqlType, String typeName) throws SQLException {
		ArrayDescriptor arrayDescriptor = new ArrayDescriptor(typeName, conn);
		ARRAY idArray = new ARRAY(arrayDescriptor, conn, ids);
		return idArray;
	}
};
```



