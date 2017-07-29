## 15.5 Simplifying JDBC operations with the SimpleJdbc classes

The`SimpleJdbcInsert`and`SimpleJdbcCall`classes provide a simplified configuration by taking advantage of database metadata that can be retrieved through the JDBC driver. This means there is less to configure up front, although you can override or turn off the metadata processing if you prefer to provide all the details in your code.

