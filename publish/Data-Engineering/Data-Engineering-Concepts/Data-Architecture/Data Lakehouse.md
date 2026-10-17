A lakehouse combines the storage of a [[Data Lake]] (open files on object storage, readable by many engines) with the table behaviour of a [[Data Warehouse]] ([[Schema On Write]], transactions). 

**What sets it apart from a data lake**: ==each table is defined by a [[Lakehouse Table Format]],== whose **metadata records the schema and the exact data files in each version**, and a write becomes visible only through one atomic commit.
> [!NOTE] To Note
> Lakehouse tables are [[Schema On Write]]. 
> Plain files stored beside them in the same lake are still [[Schema On Read]]
> ```sql
> s3://bucket/
> 	raw/orders/2026-10-03.csv      <- plain files, schema-on-read
> 	tables/orders/                 <- Iceberg table, schema-on-write
> 		data/...
> 		metadata/...
> ```
### The 4 Parts
- **Object storage:** holds the ==data and the metadata files==. Files are never changed in place; a change writes new files.
- **Table format / metadata:** records the schema and the exact list of data files in each version. It is the authority on which files are live. See [[Lakehouse Table Format]].
- **Catalog:** service that **maps each table name (e.g. `orders`) to a pointer** to its current metadata file and performs the pointer switch at commit
- **Engine (Athena, Spark etc.):** reads metadata and data, checks data against the schema, writes new files and metadata. Several engines can use the same tables.
```sql
s3://bucket/orders/ 
	data/ 
		file-a.parquet 
		file-b.parquet <- old version of some rows 
		file-c.parquet <- rewritten version of those rows 
metadata/ 
	v1.metadata.json <- lists file-a, file-b 
	v2.metadata.json <- lists file-a, file-c 

Catalog: orders -> v2.metadata.json
```
#### Reading and writing
- **Read**: the engine asks the ==catalog where the table's current metadata is (v2)==, reads the file list from it, and ==reads only those data files (file-a and file-c)==. file-b is still in the folder but is ignored.
- **Write**: 
	1. checks the **new data against the table's schema**, rejecting data that doesn't fit [[Schema On Write]]
	2. writes the new data files, 
	3. writes a new metadata file listing them, 
	4. asks the catalog to switch the pointer to the new metadata file. The switch follows the format's commit rule: it succeeds only if no other writer committed first (see [[Lakehouse Table Format]]). 
	Every write produces ==exactly one new metadata version==, however many files it adds or removes.
### Benefits 
- **Transactions:** readers see the old version or the new one, never a mix. 
- **Schema enforcement:** data that doesn't fit the table's schema is rejected at write
- **Schema changes without rewriting** and **time travel:** both come from the format's metadata (column IDs, version history). 
- **Cheap, open storage shared by many engines:** data sits in object storage in open formats, so SQL engines, Spark and ML tools use the same tables without copying data into each one. 
### Operations 
- **Maintenance jobs:** compaction (small files into larger ones), snapshot expiry, and orphan file cleanup. 
	- They must be scheduled
- **Orphan files:** files written by a job that failed before committing were never in any snapshot, so expiring snapshots never deletes them.
- **Governance:** 
	- **Permissions** are held in the catalog, per table and per column. 
	- **Storage access:** engines receive short-lived credentials for only the files a query needs, so users don't need direct access to the bucket. 
	- **Row and column rules** (e.g. hide a column, filter rows by region) are defined centrally and enforced by the engines when they read.