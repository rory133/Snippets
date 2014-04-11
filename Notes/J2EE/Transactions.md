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

When you transfer money from one account to the other, you can imagine a sequence of database accesses: the
savings account is debited using a SQL update statement, the current account is credited using a different update
statement, and a log is created in a different table to keep track of the transfer. These operations have to be done in
the same unit of work (Atomicity) because you don’t want the debit to occur but not the credit. From the perspective
of an external application querying the accounts, only when both operations have been successfully performed are
they visible (Isolation). With isolation, the external application cannot see the interim state when one account has
been debited and the other is still not credited (if it could, it would think the user has less money than she really does).
Consistency is when transaction operations (either with a commit or a rollback) are performed within the constraints
of the database (such as primary keys, relationships, or fields). Once the transfer is completed, the data can be
accessed from other applications (Durability).
