A lakehouse combines the storage of a [[Data Lake]] (open files on object storage, readable by many engines) with the table behaviour of a [[Data Warehouse]](schema-on-write, transactions). 

**What sets it apart from a data lake**: each table has metadata recording its schema and the exact data files it contains.
> [!NOTE] To Note
> Lakehouse tables are **schema-on-write**. 
> Plain files stored beside them in the same lake are still **schema-on-read**.
> ```sql
> s3://bucket/
> 	raw/orders/2026-10-03.csv      <- plain files, schema-on-read
> 	tables/orders/                 <- Iceberg table, schema-on-write
> 		data/...
> 		metadata/...
> ```
### The 3 Parts
- **Object Storage**: open columnar files, usually ==Parquet==. Never changed in place; a ==change writes new files==.
	- Write a new parquet file instead of updating it, and every write produces a new metadata version.
- ==**Table format / metadata**==: records the **schema and the exact list of data files in each version**. It is the authority on which files are live.
	- The various metadata versions are what allows for [[Time Travel]]
- **Catalog**: Pointer that maps a table name e.g. `<orders>` to it's current metadata file.
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
- Read: the engine asks the ==catalog where the table's current metadata is (v2)==, reads the file list from it, and ==reads only those data files (file-a and file-c)==. file-b is still in the folder but is ignored.
- Write: the engine writes the new data files, then a new metadata file listing them, then the ==catalog switches its pointer to the new metadata file==. Every write produces exactly one new metadata version, however many files it adds or removes.
### Where the metadata comes from 
1. **Declared by me**: schema, partitioning, location (`CREATE TABLE`; schema changes with `ALTER TABLE`). 
```sql
CREATE TABLE orders ( 
	order_id BIGINT, 
	amount DECIMAL(10,2), 
	order_date DATE 
	) 
PARTITIONED BY (month(order_date)) 
LOCATION 's3://bucket/orders/' 
TBLPROPERTIES ('table_type' = 'ICEBERG');
```
2. Derived by the engine: file list, row counts, min/max per column, versions. Every write produces a new metadata file, then the catalog pointer switches to it.
### Benefits
- Schema changes without rewriting: `ALTER TABLE` adds, renames or drops a column as a metadata change. The old data files stay as they are.
- Old metadata versions list the old data files, so the table can be queried as it was earlier, until cleanup removes them.
