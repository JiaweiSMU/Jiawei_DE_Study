- Schema is ==applied once when the data is written==. Every row is checked and converted with bad data being stopped
Mainly applicable to [[ETL (Extract, Transform, Load)]] when the data is being loaded

### Steps
1. Firstly you will define the schema and create the table
```sql
CREATE TABLE orders (order_id BIGINT NOT NULL, order_date DATE, amount DECIMAL(12,2), region VARCHAR(10)
```
2. During the load, the data is checked against the schema.
	- e.g. if we are trying to insert into amount but the column value is 'abc' instead of 10.20, it's rejected
Benefits:
- Reads are fast since the data is already parsed and typed
- Reads are trustworthy, since we know the value fits the data type that we declare
- Bad data is found at load instead of letting someone else find it
Cons:
- Need to know the shape of the data before loading
- Quite difficult to make changes to the schema

