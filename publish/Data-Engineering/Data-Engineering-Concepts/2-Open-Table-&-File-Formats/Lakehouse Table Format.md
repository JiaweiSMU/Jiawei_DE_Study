It's a specification for **==metadata==** that ==records a table's schema and exactly which data files make up each version==, plus the rules for changing the table: how to commit a new version atomically, how to record changed rows, and how to evolve the schema.

Each ==write produces one new version== (metadata), and the ==catalog points to the current one== (metadata). How this fits with storage, the catalog and engines: see [[Data Lakehouse]].
### What the metadata records
For each version of the table:
- **Schema:** column names and types, each column with an internal ID
- **Partitioning:** e.g. partition by month of `order_date`
- **File list:** the exact data files in this version
- **Per-file statistics:** row count, and min/max per column, so the engine can skip files without opening them
- **Version history:** the list of earlier versions (snapshots)
### Where the metadata comes from
1. **Declared by me:** schema, partitioning, location (`CREATE TABLE`; schema changes with `ALTER TABLE`).
```sql
CREATE TABLE orders (
    order_id   BIGINT,
    amount     DECIMAL(10,2),
    order_date DATE
)
PARTITIONED BY (month(order_date)) --order_date is in the column list, partition values come from data itself, NOT folder names
LOCATION 's3://bucket/orders/'
TBLPROPERTIES ('table_type' = 'ICEBERG');
```
2. **Derived by the engine on each write:** file list, row counts, min/max per column, versions. Every write produces a new metadata file, then the catalog pointer switches to 
### The rules
> [!NOTE] Design principle
> ==Data files are immutable== because **object storage and Parquet** don’t allow in-place changes. A table format builds on that: its versions, snapshot reads, time travel and file statistics are all valid only because listed files never change. 
> ==Every change is therefore made by writing new files and a new metadata version.==
1. [[Commit]] (How a new version becomes current)
	- Write **new data files**, then a **new metadata file** listing them, then **switch** the catalog pointer to it, but ==**only if the pointer still points to the version the write started from**==. If another writer committed first, the switch fails and the write retries on the newer version.
2. **Changing rows** (How Updates & Deletes work without editing files)
	1. [[Copy On Write]]: Read the file where you need to update the row, write that whole file again + the row being changed and commit that.
	2. [[Merge On Read]]: write small delete files that mark removed rows; readers apply them while reading.
3. [[Schema Evolution]] (How columns change without rewriting files)
	- ==Columns are tracked by ID==, not by name or position. Adding, renaming or dropping a column changes only the metadata.
### What this enables
- **Time travel:** old metadata versions still list their data files, so the table can be queried as it was at an earlier version, until those versions are expired.
### Downsides
- Frequent small writes (e.g. a job committing every minute) creates many small data files, and each commit also adds a snapshot and a manifest.
	- Query planning slows down because there is more metadata to read.
	- **Compaction is needed** to rewrite small data files and manifests into larger ones.
- Old snapshots keep replaced files in storage.
	- Snapshots must be expired before those files can be deleted.
	- Once a snapshot is expired, **time travel to that version is no longer possible**.
- Compaction and snapshot expiry do not run by themselves. They are jobs one must schedule.
Running compaction and expiry is an operational task: see Operations in [[Data Lakehouse]].