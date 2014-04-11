### Overview
* Since Java EE 7, Managed Beans can also have declarative transactions in the same way.
* ACID: Atomicity, Consistency, Isolation, and Durability.


#### ACID
| Term |Description |
| ----- | ------ |
| Atomicity  | A transaction is composed of one or more operations grouped in a unit of work. At the conclusion of the transaction, either these operations are all performed successfully (commit) or none of them is performed at all (rollback) if something unexpected or irrecoverable happens.  |
| Consistency  | At the conclusion of the transaction, the data are left in a consistent state. |
| Isolation | The intermediate state of a transaction is not visible to external applications. |
| Durability | Once the transaction is committed, the changes made to the data are visible to other applications.|
