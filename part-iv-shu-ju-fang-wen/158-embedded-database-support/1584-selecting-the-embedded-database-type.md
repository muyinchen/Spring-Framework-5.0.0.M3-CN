#### 使用HSQL

spring支持HSQL 1.8.0及以上版本。HSQL是缺省默认内嵌数据库类型。如果显式指定HSQL，设置`embedded-database` Tag的`type`属性值为`HSQL`。如果使用builderAPI.调用`EmbeddedDatabaseType.HSQL`的`setType(EmbeddedDatabaseType)`方法。

#### 使用H2

Spring也支持H2数据库。设置`embedded-database` tag的`type`类型值为`H2`来启用H2。如果你使用了builder API。调用`setType(EmbeddedDatabaseType)` 方法设置值为`EmbeddedDatabaseType.H2`。

#### 使用Derby

Spring也支持 Apache Derby 10.5及以上版本，设置`embedded-database` tag的`type`属性值为`DERBY`来开启DERBY。如果你使用builder API，调用`setType(EmbeddedDatabaseType)`方法设置值为`EmbeddedDatabaseType.DERBY`.

