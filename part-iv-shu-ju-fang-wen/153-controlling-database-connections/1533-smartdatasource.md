### 15.3.3 SmartDataSource

The`SmartDataSource`interface should be implemented by classes that can provide a connection to a relational database. It extends the`DataSource`interface to allow classes using it to query whether the connection should be closed after a given operation. This usage is efficient when you know that you will reuse a connection.

