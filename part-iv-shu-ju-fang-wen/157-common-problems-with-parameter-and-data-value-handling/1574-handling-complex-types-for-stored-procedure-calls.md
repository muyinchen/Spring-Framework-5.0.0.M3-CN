### 15.7.4**处理存储过程调用的复杂类型**

当你调用存储过程时有时需要使用数据库特定的复杂类型。为了兼容这些类型，当存储过程调用返回时Spring提供了一个`SqlReturnType` 来处理这些类型，`SqlTypeValue`用于存储过程的传入参数。

下面是一个用户自定义类型`ITEM_TYPE`的Oracle `STRUCT`对象的返回值例子。`SqlReturnType`有一个方法`getTypeValue`必须被实现。而这个接口的实现将被用作`SqlOutParameter`声明的一部分。

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

你可以使用`SqlTypeValue `类往存储过程传入像`TestItem`那样的Java对象。你必须实现`SqlTypeValue`接口的`createTypeValue`方法。你可以使用传入的连接来创建像`StructDescriptors`这样的数据库指定对象,或`ArrayDescriptors`。下面是相关的例子。

```java
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

`SqlTypeValue`会加入到包含输入参数的Map中，用于执行存储过程调用。

`SqlTypeValue` 的另外一个用法是给Oracle的存储过程传入一个数组。Oracle内部有它自己的`ARRAY`类，在这些例子中一定会被使用，你可以使用`SqlTypeValue`来创建Oracle `ARRAY`的实例，并且设置到Java `ARRAY`类的值中。

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



