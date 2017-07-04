## 15.1Introduction to Spring Framework JDBC

The value-add provided by the Spring Framework JDBC abstraction is perhaps best shown by the sequence of actions outlined in the table below. The table shows what actions Spring will take care of and which actions are the responsibility of you, the application developer.

**Table15.1.Spring JDBC - who does what?**

| Action | Spring | You |
| :--- | :--- | :--- |
| Define connection parameters. |  | X |
| Open the connection. | X |  |
| Specify the SQL statement. |  | X |
| Declare parameters and provide parameter values |  | X |
| Prepare and execute the statement. | X |  |
| Set up the loop to iterate through the results \(if any\). | X |  |
| Do the work for each iteration. |  | X |
| Process any exception. | X |  |
| Handle transactions. | X |  |
| Close the connection, statement and resultset. | X |  |

The Spring Framework takes care of all the low-level details that can make JDBC such a tedious API to develop with.

  


