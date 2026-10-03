> [!NOTE]
> Prevent partial writes and corrupted states. All changes must be successfully committed or rolled backed.
## Atomicity: All or nothing
**Each statement in a transaction** (to read, write, update or delete data) is treated as a single unit.
Either all of its statements ==take effect together or none do==.
- If a transaction fails or the database crashes midway, none of the changes survive. The database won't be in a half-written state.
## Consistency: Declared rules always hold
Transactions only ==make changes to tables in predefined and predictable ways==.
So that corruption or error in data, does not create unintended consequences on the integrity of the table.
- Primary keys, Foreign keys, unique, not-null and check constraints are true before and after every transaction
	- The DB will check the constraints and abort the transaction if there are any violation
- **Limit**: It only enforces the rules that are declared (e.g. If we write `Column y` can't be negative, it will then check it. If not it won't check it)
## Isolation: Overlapping transactions don't interfere
Ensures that when multiple users are reading and writing from the same table at once, their ==transactions don't interfere with or affect one another to a degree== chosen by the `Isolation Level`
### Isolation Levels
> [!NOTE] Locking
> A lock makes other transactions wait before doing something conflicting to the same row. 
> - Shared lock (for reading) makes writers wait but lets other readers through.
> -  Exclusive lock (for writing) makes other writers wait

These are setting on a transaction that will decide how much interference from other concurrent transactions it will tolerate.
- There are trade-offs to be made at each Isolation level

| Level                | Guarantee                                         | Anomalies still allowed                  | Lock strategy (lock-based engines)                                                                             | Use when                                                                         |
| -------------------- | ------------------------------------------------- | ---------------------------------------- | -------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------- |
| **Read Uncommitted** | None on reads                                     | Dirty read, non-repeatable read, phantom | No shared lock taken to read, so reads never wait for writers                                                  | When exact **accuracy is not as important**                                      |
| **Read Committed**   | Only committed data is read                       | Non-repeatable read, phantom             | Shared lock taken on read, released as soon as the read finishes                                               | When transactions **require the data to be consistent** throughout the execution |
| **Repeatable Read**  | Rows you read don't change under you              | Phantom                                  | Shared lock held until ==commit or rollback==. New matching rows can still be inserted                         | A multi-statement transaction needs the same view throughout                     |
| **Serializable**     | Outcome equals running transactions one at a time | None                                     | Shared locks plus locks on the key ranges scanned, held until the end. ==New matching rows can't be inserted== | A write depends on a check across rows the transaction doesn't modify            |
- **Dirty read** 
	- Definition: T1 reads data T2 has written but not committed
	- Example: T2 sets address to "B" without committing. T1 reads "B". T2 rolls back, so T1 used a value that never officially existed 
	- First prevented at: Read Committed
- **Non-repeatable read** 
	- Definition: T1 reads the same row twice and gets different values
	- Example: T1 reads balance 100. T2 sets it to 50 and commits. T1 reads again and gets 50 - First prevented at: Repeatable Read
- **Phantom read**
	- Definition: T1 runs the same query twice and gets a different set of rows
	- Example: T1 counts 10 orders. T2 inserts one and commits. T1 counts 11 - First prevented at: Serializable
## Durability: Committed means it survives a crash
Ensures that changes to the data made successfully by completed transactions will be saved, even in the event of a system crash
- Once the data is `COMMIT`, ==it will survive a crash==
