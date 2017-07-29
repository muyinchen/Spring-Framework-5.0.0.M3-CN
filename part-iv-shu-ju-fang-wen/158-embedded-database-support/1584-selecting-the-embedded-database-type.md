#### Using HSQL

Spring supports HSQL 1.8.0 and above. HSQL is the default embedded database if no type is specified explicitly. To specify HSQL explicitly, set the`type`attribute of the`embedded-database`tag to`HSQL`. If you are using the builder API, call the`setType(EmbeddedDatabaseType)`method with`EmbeddedDatabaseType.HSQL`.

#### Using H2

Spring supports the H2 database as well. To enable H2, set the`type`attribute of the`embedded-database`tag to`H2`. If you are using the builder API, call the`setType(EmbeddedDatabaseType)`method with`EmbeddedDatabaseType.H2`.

#### Using Derby

Spring also supports Apache Derby 10.5 and above. To enable Derby, set the`type`attribute of the`embedded-database`tag to`DERBY`. If you are using the builder API, call the`setType(EmbeddedDatabaseType)`method with`EmbeddedDatabaseType.DERBY`.

