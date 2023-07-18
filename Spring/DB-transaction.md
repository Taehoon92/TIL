# DB - Transaction

Transaction is a sequence of multiple operations performed on a database and all of operations works with single logical unit. There is never a case that only half of the operations are performed and the results saved. The typical example for understanding transaction is transferring money. For example, suppose there is a transaction to transfer $5 from A to B. This action can be broken down into the following operations:  

1. Subtract $5 from the balance of Account A.
2. Add $5 to the balance of Account B.

In this process, these two operations must operate as one operation. If operation 1 succeeds, and operation 2 fails, a critical problem occurs in which the balance of Account A decreases even though the transferring failed. If you use a transaction, you can save all operations to the DB when they succeed, and return to the inital state if they fail.  

All tasks are successfully reflected in the database is called a `commit`. The process that undoing changes made by a transaction is called a `rollback`.

---

## Transaction ACID

Transaction must be atomic, consistent, isolated and durable. These properties are abbreviated as **ACID**.  
* **Atomicity** : Operations executed within transaction must all succeed or all fail as if they were a single operation.
* **Consistency** : All transactions must maintain a consistent database status. For example, the integrity constraints set by the database must always be satisfied.
* **Isolation** : It isolates executing transactions at the same time, so that they do not affect each other. It can be set Isolation level because of performance issue.
* **Durability** : A successful transaction commit survive permanently. The result must always be recorded after transaction succeed. An entry is added to the database transaction log for each successful transaction.

