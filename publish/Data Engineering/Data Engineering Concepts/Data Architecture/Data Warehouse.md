A data warehouse is the integrated, historical copy of data from many sources, stored on an engine built for [[OLAP (Online Analytical Processing)]] workloads
- Data is optimized for users to query
### How data gets in 
Data is copied from the sources by scheduled batch extract or by change data capture. It first lands in `Staging` as raw, unmodified copies, one per source. It is then transformed (keys matched across sources, names and types made consistent, history kept) into the warehouse tables.

There are two approaches, which differ in where the transformation runs: 
1. **ETL** (Extract, Transform, Load): ==staging lives outside the warehouse==. An external engine transforms the data, and ==only the result is loaded==. 
2. **ELT** (Extract, Load, Transform): ==raw data is loaded into a staging schema inside the warehouse==, and the warehouse engine transforms it with SQL. 
### Modelling approaches 
1. [[Inmon]]: build one normalized enterprise warehouse first, then derive data marts from it. 
2. [[Kimball]]: build star-schema marts per business process, tied together by shared dimensions.
### Structure in a Data Warehouse (schema-on-write)
In a data warehouse, structure is defined before loading and enforced while loading.
#### The steps 
1. **Define**: the table is declared with `CREATE TABLE` (column names, types, rules such as `NOT NULL`). 
2. **Load**: every row is checked against the table. A row that does not fit is rejected and the load fails. 
3. **Store**: the engine converts the data into its own storage format. Queries read that, not the original files.
Example: in Redshift I run `CREATE TABLE` first, then `COPY` the files in from S3.
#### When data don't match the table
The mismatch is found at `load time`. It never appears during query time. The load will just fail.