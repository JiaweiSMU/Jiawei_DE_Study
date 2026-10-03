A workload where queries aggregate ==over millions of rows but read only a few columns. ==(OLTP reads or writes a few whole rows by key.)
## Characteristics 
- **Operations:** aggregations (`SUM`, `COUNT`, `AVG`) with filters and `GROUP BY` over a large share of the table. 
- **Writes are large batch appends,** not single-row updates. Compressed column blocks are immutable, so an update is a delete marker plus a new row, cleaned up later (`VACUUM` in Redshift).
* **Data source:** copied from OLTP systems and other sources (logs, events, files) by batch extract or change data capture. 
## Column layout 
```sql
amount column (500M values) 
Block 1: positions 1 to 250,000 min=5 max=980 
Block 2: positions 250,001 to 500,000 min=10 max=1,200 
Block 3: positions 500,001 to 750,000 min=5 max=870 
... 
Block 2,000: last 250,000 positions min=8 max=1,050 

order_date column (500M values) 
Block 1: min=2021-01-01 max=2021-01-03 
Block 2: min=2021-01-03 max=2021-01-06
```
- Each column is stored separately and split into blocks. A block holds many values of one column and records their min and max.
- ==All columns keep the same row order, so position `n` in every column belongs to the same row==
- **Good for aggregation:** a query reads **only the columns it uses** (aggregated, filtered, and grouped). All other columns are never touched.
- **Filtering:** the engine checks each block's min/max and skips blocks that cannot contain a match. It then reads only the matching row positions from the other columns.
* **Bad for point operations:** fetching one full row means reading and decompressing a whole block per column. 
## [[Data Modelling]] 
* Usually dimensional. A [[Star Schema]] has a narrow fact table (keys and measures) surrounded by denormalized dimension tables. A [[Snowflake Schema]] is a star with the dimensions normalized.



Think of it as a massive database that stores OLTP data, for processing and data analysis.
- Uses a [[Data Warehouse]] instead
- Tables are usually not [[Normalized]]
- Their queries are usually Read heavy and not much write happens
