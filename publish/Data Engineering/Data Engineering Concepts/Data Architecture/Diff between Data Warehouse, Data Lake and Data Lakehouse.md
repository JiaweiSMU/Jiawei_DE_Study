![[Pasted image 20261003150123.png]]
- [[Data Warehouse]]: Data must fit the table's declared schema and it can't hold unstructured data.
- [[Data Lake]]: It is able to hold any file format and only when the data is read, then structure is applied. 
- [[Data Lakehouse]]: A data lake and a table format (`Iceberg, Delta Lake`), The table format is a set of metadata files, that records which data files make up a table at each version.

|                          | [[Data Warehouse]]                              | [[Data Lake]]                              | [[Data Lakehouse]]                                                |
| ------------------------ | ----------------------------------------------- | ------------------------------------------ | ----------------------------------------------------------------- |
| **What it is**           | Integrated, modelled tables built for analytics | Files of any kind on object storage        | Lake storage with warehouse-style tables on top                   |
| **Data it holds**        | Structured tables (plus JSON columns)           | Anything: CSV, JSON, Parquet, images, logs | Tables as open files (`parquet`), with any other file beside them |
| **Storage**              | The engine's own format                         | Object storage, any format                 | Object storage, open format (usually Parquet)                     |
| **Structure applied**    | On write                                        | On read                                    | On write, for tables                                              |
| **A bad file is caught** | At load                                         | At query time, or never                    | At write (When incoming rows are checked against the schema)      |
| **What defines a table** | The engine                                      | A folder plus a **catalog** entry          | **Metadata** listing the exact files                              |
