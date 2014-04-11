### Overview
* Since Java EE 7, Managed Beans can also have declarative transactions in the same way.
* ACID: Atomicity, Consistency, Isolation, and Durability.
* To have a reliable transaction across several resources, the transaction manager needs to use an XA (eXtended
Architecture) resource manager interface. XA is a standard specified by the Open Group (www.opengroup.org)
for distributed transaction processing (DTP) that preserves the ACID properties. It is supported by JTA and allows
heterogeneous resource managers from different vendors to interoperate through a common interface. XA uses
a two-phase commit (2pc) to ensure that all resources either commit or roll back any particular transaction
simultaneously.
* The EJB model was designed to manage transactions. In fact, transactions are natural to
EJBs, and by default each method is automatically wrapped in a transaction. This default behavior is known as a
container-managed transaction (CMT), because transactions are managed by the EJB container (a.k.a. declarative
transaction demarcation). You can also choose to manage transactions yourself using bean-managed transactions
(BMTs), also called programmatic transaction demarcation. Transaction demarcation determines where transactions
begin and end.


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

#### Read Conditions
Transaction isolation can be defined using different read conditions (dirty reads, repeatable reads, and phantom
reads). They describe what can happen when two or more transactions operate on the same data at the same time.
Depending on the level of isolation you put in place you can totally avoid or allow concurrent access.
* Dirty reads: Occur when a transaction reads uncommitted changes made by the previous transaction.
* Repeatable reads: Occur when the data read are guaranteed to look the same if read again during the same transaction.
* Phantom reads: Occur when new records added to the database are detectable by transactions that started prior to the insert. Queries will include records added by other transactions after their transaction has started.

#### Transaction Isolation Levels
Databases use several different locking techniques to control how transactions access data concurrently. Locking
mechanisms impact the read condition described previously. Isolation levels are commonly used in databases to
describe how locking is applied to data within a transaction. The four types of isolation levels are:
* Read uncommitted (less restrictive isolation level): The transaction can read uncommitted data. Dirty, nonrepeatable, and phantom reads can occur.
* Read committed: The transaction cannot read uncommitted data. Dirty reads are prevented but not nonrepeatable or phantom reads.
* Repeatable read: The transaction cannot change data that are being read by a different transaction. Dirty and nonrepeatable reads are prevented but phantom reads can occur
* Serializable (most restrictive isolation level): The transaction has exclusive read. The other transactions can neither read nor write the same data.

Generally speaking, as the isolation level becomes more restrictive the performance of the system decreases because
transactions are prevented from accessing the same data. However, the isolation level enforces data consistency. Note that not all RDBMS (Relational Database Management Systems) implement these four isolation levels.

#### Container managed transaction attributes.
* REQUIRED. This attribute (default value) means that a method must always be invoked within a transaction. The container creates a new transaction if the method is invoked from a nontransactional client. If the client has a transaction context, the business method runs within the client’s transaction. You should use REQUIRED if you are making calls that should be managed in a transaction, but you can’t assume that the client is calling the method from a transaction context.
* REQUIRES_NEW. The container always creates a new transaction before executing a method, regardless of whether the client is executed within a transaction. If the client is running within a transaction, the container suspends that transaction temporarily, creates a second one, commits or rolls it back, and then resumes the first transaction. This means that the success or failure of the second transaction has no effect on the existing client transaction. You should use REQUIRES_NEW when you don’t want a rollback to affect the client.
* SUPPORTS. The EJB method inherits the client’s transaction context. If a transaction context is available, it is used by the method; if not, the container invokes the method with no transaction context. You should use SUPPORTS when you have read-only access to the database table.
* MANDATORY. The container requires a transaction before invoking the business method but should not create a new one. If the client has a transaction context, it is propagated; if not, a javax.ejb.EJBTransactionRequiredException is thrown.
* NOT_SUPPORTED. The EJB method cannot be invoked in a transaction context. If the client has no transaction
context, nothing happens; if it does, the container suspends the client’s transaction, invokes
the method, and then resumes the transaction when the method returns.
* NEVER. The EJB method must not be invoked from a transactional client. If the client is running within a transaction context, the container throws a javax.ejb.EJBException.

![Image](../master/Notes/Images/Transactions_1.PNG)
