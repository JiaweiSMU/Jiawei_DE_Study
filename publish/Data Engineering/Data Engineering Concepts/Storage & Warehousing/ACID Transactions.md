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
Ensures that when multiple users are reading and writing from the same table at once, their ==transactions don't interfere with or affect one another==.
### Isolation Levels
Setting on a transaction that will decide how much interference from other concurrent transactions it will tolerate.
- There are trade-offs to be made at each Isolation level
![[Pasted image 20261002233344.png]]

#### Read phenomena
- **Dirty Read**: When another transaction can read data that has yet been written but not committed
	1. T2 updates a customer's address to "B" and has not committed.
	2. T1 reads the address and gets "B".
	3. T2 rolls back. T1 acted on a value that never officially existed.
- **Non-Repeatable Read**: When a transaction reads the same row twice but gets different value each time
	1. T1 reads balance = 100.
	2. T2 updates the balance to 50 and commits.
	3. T1 reads again and gets 50.
- **Phantom Read**: When a transaction runs the same query twice and gets a different set of rows
	1. T1 counts orders for today and gets 10.
	2. T2 inserts a new order and commits.
	3. T1 counts again and gets 11. No row T1 saw has changed; a new one appeared.



## Durability: Committed means it survives a crash
Ensures that changes to the data made successfully by completed transactions will be saved, even in the event of a system crash
- Once the data is ==`COMMIT`, it will survive a crash==
