### Using AS
```sql
SELECT name AS full_name
SELECT * FROM users AS u
```
`AS` renames a column or table in the output - the first example relabels a column, the second gives the table itself a shorter alias to reference elsewhere in the query.
### Using Expressions in SELECT
An expression is a combination of values, operators, and functions that the database evaluates to produce a result - you're not limited to selecting raw columns.
```sql
SELECT
    price + tax AS total_price,
    quantity * unit_price AS total_cost
FROM orders
```
