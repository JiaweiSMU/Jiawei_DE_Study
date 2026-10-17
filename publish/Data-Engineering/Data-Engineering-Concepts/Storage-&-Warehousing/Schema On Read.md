- Data is stored exactly as written, unchecked. (No validation when storing)
- The ==schema is only applied when a query reads the data==. 
This is mainly for [[ELT (Extract, Load, Transform)]]
### Structure in a Data Lake 
In a data lake, structure is applied only when the data is read. Nothing is validated when files are written.
#### The 3 parts
It always follows a similar pattern in other systems.
1. **Storage**: holds files and does not inspect them (e.g. S3).
2. ==**Catalog**:== holds the ==table definitions== (e.g. Glue Data Catalog, Hive Metastore)
3. **Query engine**: looks up the definition and interprets the files with it (e.g. Athena, Trino, Spark).
#### The table definition 
It ==**must exist before the files can be queried**==. In AWS it is created with `CREATE EXTERNAL TABLE` or by a Glue crawler. It declares: 
 - Column names and types
 - File format
 - Location: the root folder. The definition covers all files under it, including files added later.
 - Partition columns
#### When a file does not match the definition 
The **mismatch is found only at query time**. The query may fail, return `NULL`, or silently read wrong values