### 13.5.5&lt;tx:advice/&gt;settings

This section summarizes the various transactional settings that can be specified using the`tag. The default`settings are:

* [Propagation setting](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/transaction.html#tx-propagation)is`REQUIRED.`

* Isolation level is`DEFAULT.`

* Transaction is read/write.

* Transaction timeout defaults to the default timeout of the underlying transaction system, or none if timeouts are not supported.

* Any`RuntimeException`triggers rollback, and any checked`Exception`does not.

You can change these default settings; the various attributes of the`tags that are nested within`and\`\`tags are summarized below:

**Table13.1. settings**

| Attribute | Required? | Default | Description |
| :--- | :--- | :--- | :--- |
| `name` | Yes |  | Method name\(s\) with which the transaction attributes are to be associated. The wildcard \(_\) character can be used to associate the same transaction attribute settings with a number of methods; for example,\`get_`,`handle\_`,`on\_Event\`, and so forth. |
| `propagation` | No | REQUIRED | Transaction propagation behavior. |
| `isolation` | No | DEFAULT | Transaction isolation level. |
| `timeout` | No | -1 | Transaction timeout value \(in seconds\). |
| `read-only` | No | false | Is this transaction read-only? |
| `rollback-for` | No |  | `Exception(s)`that trigger rollback; comma-delimited. For example,`com.foo.MyBusinessException,ServletException.` |
| `no-rollback-for` | No |  | `Exception(s)`that do\_not\_trigger rollback; comma-delimited. For example,`com.foo.MyBusinessException,ServletException.` |



