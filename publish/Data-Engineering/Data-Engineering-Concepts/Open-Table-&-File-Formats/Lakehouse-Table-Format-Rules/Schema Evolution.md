To make changes to a table's column after data already exists
- Adding, Renaming, Reordering or Widening a type

Each **column** when it's created is ==assigned a ID that never changes and is never reused==.
Readers match column by ID. Making it easy to make changes to a column name.
### How it works
**Schema v1:**
```text
ID 1 → order_id   bigint
ID 2 → amount     decimal(12,2)
ID 3 → order_date date
```
`file-1.parquet` is written under v1, so it contains columns 1, 2 and 3.

**Two changes are made:**
```sql
ALTER TABLE orders RENAME COLUMN amount TO amount_sgd;
ALTER TABLE orders ADD COLUMN channel STRING;
```
Each change is a commit: a ==new metadata version with a new schema==. No data file is touched.

**Schema v2:**
```text
ID 1 → order_id    bigint
ID 2 → amount_sgd  decimal(12,2)   ← same ID, new name
ID 3 → order_date  date
ID 4 → channel     string          ← new ID
```
`file-2.parquet` is written after the change, so it contains columns 1, 2, 3 and 4.

When querying the table, it will look up the IDs in the current schema.
![[Pasted image 20261017205158.png]]
- New column is `NULL` for old rows.

### Each kind of change

| Change                                         | What happens                                                                                               | Why it’s safe                                       |
| ---------------------------------------------- | ---------------------------------------------------------------------------------------------------------- | --------------------------------------------------- |
| **Add** a column                               | Gets a new ID. Old files lack it, so old rows read `NULL`.                                                 | Old files are simply missing that ID                |
| **Rename**                                     | Only the name for that ID changes                                                                          | Readers never use names to find data                |
| **Drop**                                       | The ID is removed from the schema. Old files still hold the data physically, but nothing asks for that ID. | The data is hidden, not deleted                     |
| **Drop, then add a column with the same name** | The new column gets a **new** ID                                                                           | The old values can’t come back under the new column |
| **Reorder**                                    | Only the column order in the schema changes                                                                | Readers don’t use position                          |
| **Widen a type**                               | Allowed only for safe changes: `int → bigint`, `float → double`, larger decimal precision                  | Every old value is still valid under the new type   |
Some changes are not allowed:
- **Unsafe type changes are refused**, ==such as `string → int`==. 
- **A new column must allow `NULL`**, because old rows have no value for it.