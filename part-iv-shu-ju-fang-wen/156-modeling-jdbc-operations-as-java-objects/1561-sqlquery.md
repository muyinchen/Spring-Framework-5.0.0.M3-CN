### 15.6.1 SqlQuery

`SqlQuery`类主要封装了SQL查询，本身可重用并且是线程安全的。子类必须实现`newRowMapper(...)`方法，这个方法提供了一个`RowMapper`实例，用于在查询执行返回时创建的结果集迭代过程中每一行映射并创建一个对象。`SqlQuery`类一般不会直接使用；因为`MappingSqlQuery`子类已经提供了一个更方便从列映射到Java类的实现。其他继承`SqlQuery`的子类有`MappingSqlQueryWithParameters`和`UpdatableSqlQuery`。

