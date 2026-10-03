> [!NOTE] 
> It's similar to Iceberg itself, with the main difference being:
> 1. Within the metadata, `delta` writes down what changed, while `Iceberg` writes down what the table is

It's a form of a table format. With a set of rules for a directory of [[Parquet]] files and a `_delta_log/` folder.
- Used to build a[[Data Lakehouse]] architecture on top of a [[Data Lake]]
### Delta Table
A Delta table is one directory holding two things: Parquet data files, and a log (`_delta_log/`) that records which of those files make up the table.
```sql
s3://bucket/orders/
  part-a1f3.parquet          <- data files
  part-b7c2.parquet
  part-c9d4.parquet
  _delta_log/                <- the log
    00000000000000000000.json
    00000000000000000001.json
    00000000000000000002.json
```
#### Delta Log
The log is Delta Lake's metadata. Iceberg keeps the same kind of information in its `metadata/` folder.  
- Each file in `_delta_log/` is one commit, and ==each commit is one version of the table==. The file name is the version number.
- A commit lists the ==files added and the files removed==, and ==the schema when it is set or changed==.
- The table is defined by the log: the engine replays the commits in order, and the table is every file added and not later removed.
- ==Log files are never edited==. Every change to the table is a new commit.
```sql
		-- Version 0 (Create Table)
		{"protocol": {"minReaderVersion": 1, "minWriterVersion": 2}}
		{"metaData": {"schemaString": "order_id long, region string, amount decimal", "partitionColumns": []}}
		
		-- Version 1 (Insert records)
		{"add": {"path": "part-a1f3.parquet", "size": 104857600, "stats": {"numRecords": 1000000, "minValues": {"order_id": 1}, "maxValues": {"order_id": 1000000}}}}
		{"add": {"path": "part-b7c2.parquet", "size": 104857600, "stats": {"numRecords": 1000000, "minValues": {"order_id": 1000001}, "maxValues": {"order_id": 2000000}}}}
```
#### Data Files
- A Parquet file is written once and never modified. A change to a row writes a new [[Parquet]] file. 
- A file is part of the table only if the log references it (an `add` with no later `remove`). A file that is only present in the directory is not in the table.
- Removed files stay on S3 until `VACUUM` deletes them (7 days by default). Until then, older versions can still be read.
