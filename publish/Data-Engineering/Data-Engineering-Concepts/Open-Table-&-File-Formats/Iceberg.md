A table format for large analytical tables in a [[Data Lake File Format]]. It is fast because the engine never lists folders
- The metadata names every data file and stores each file's min/max
- Files that cannot match a query are skipped without being opened.
> [!NOTE] Importance of ID for Schema Evolution
> In a Data Lake, Columns are matched by Name (Parquet) or Position (CSV). But both are not durable.
> - Rename `amount` to `total` with name matching: old files have no column called `total`, so every old row returns `NULL`.
>- Drop a middle column with position matching: every later column shifts by one, and values are read into the wrong columns.
### What it provides
* **Transactions**
	* Every write is ==all-or-nothing==. The engine writes the new files first, then the catalog switches to the new metadata in one step.
	* Readers see the table either before the write or after it, never in between.
- **Schema evolution**
	- Columns can be added, dropped, renamed and re-ordered without rewriting the table.
	- **How**: ==each column has a permanent ID, and data files store values under the ID==. A rename only changes the name attached to that ID in the metadata. 
		```
		Metadata (schema)              file1.parquet (footer)
		  ID 1 → order_id                ID 1: 101, 102
		  ID 2 → amount                  ID 2: 50, 20
		  ID 3 → region                  ID 3: SG, MY
		```
* **Time travel and rollback** 
	* ==Every write creates a snapshot== (a version of the table). Old snapshots stay readable until they are expired.
	* **Time travel**: query the table as it was at an earlier snapshot, by **snapshot ID or timestamp**.
	* **Rollback**: **point** the table back to an **earlier snapshot** after a bad write. No data is rewritten.
- Partition declared as a function of a real column
```sql
CREATE TABLE orders (
	order_id bigint, 
	order_ts timestamp, 
	amount double
) 
PARTITIONED BY (day(order_ts)) 
LOCATION 's3://lake/orders/' 
TBLPROPERTIES ('table_type' = 'ICEBERG');
```
- On write, the engine computes `day(order_ts)` and records it for each file in the metadata. No `order_day` column exists.
- On read, a filter on `order_ts` is converted to the matching days, and files of other days are skipped.
