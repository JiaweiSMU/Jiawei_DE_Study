**Subquery**: a query written inside another query. The inner query finds a value (or list of values), and the outer query uses it.

### Subqueries for filtering: Which operator to use
The right operator depends on **what the subquery returns**.
#### 1. Subquery returns one value → use `=`, `>`, `<`, etc.
Aggregates like `AVG`, `MAX` or `MIN` ==(with no `GROUP BY`)== give back a single value, so you can compare against it directly.
```sql
-- Find customers who spent more than the average 
SELECT name 
FROM customers 
WHERE spend > (SELECT AVG(spend) FROM customers); 
			-- └── inner query: works out the average first
```
⚠️ If the subquery returns more than one row, `=` or `>` will throw an error.
#### 2. Subquery returns a list of values → use `IN` / `NOT IN`
`IN` checks whether a value is in the list.
```sql
-- Find employees in departments that have high earners
SELECT
  first_name,
  last_name,
  department
FROM employees
WHERE department IN (
  SELECT DISTINCT department
  FROM employees
  WHERE salary > 100000
);
```
⚠️ With `NOT IN`, if the list contains even one NULL, the query returns **zero rows**. Use `NOT EXISTS` instead.
#### 3. Only checking whether a match exists → use `EXISTS` / `NOT EXISTS`
`EXISTS` checks, for **each row in the outer query**, whether **at least one** matching row exists in the subquery. It returns true/false and **stops at the first match**. The subquery can look at another table or the same table.
> [!NOTE] 
> `EXISTS` only returns **true / false**. Whatever the subquery selects is thrown away and never reaches the outer query. That's why the convention is `SELECT 1`: the values don't matter.

```sql
-- Customers who HAVE placed at least one order 
SELECT 
c.name 
FROM customers c 
WHERE EXISTS 
	( SELECT 1 FROM orders o WHERE o.customer_id = c.id );
	-- Use return 1 since outer query can't use the values
```

SQL goes through `customers` ==one row at a time and asks==, "Is there any row in `orders` for this customer?![[Pasted image 20260927001618.png]]

### Subqueries in `SELECT`
For each row, it only produces a single `Row & Column`
- If more than 1 row is returned, an error is produced. Some aggregate (`MAX, SUM, COUNT`) is required to guarantee it returns a single row.
#### 1. Uncorrelated
Where subquery does not reference the outer query, it's only ran once and the result of the subquery is re-used
```sql
SELECT 
	country, arrivals, 
	arrivals 
		* 100.0 / (SELECT SUM(arrivals) FROM visitor_arrivals) AS pct_of_total 
	FROM visitor_arrivals;
```
#### 2. Correlated
Subquery ==references a column from the outer query==, so the result changes row by row
```sql
SELECT 
	c.country, c.region, 
	(SELECT MAX(v.arrival_date) FROM visitor_arrivals v 
		WHERE v.country = c.country) AS last_arrival 
FROM countries c;
```

### Subqueries in `FROM`
> [!NOTE] 
> - Inner Subquery MUST have an Alias (e.g. AS table_1)
> - Any derived tables can be rewritten as a CTE as well

Acts like a **derived table**
- `Inner Subquery`: produces a **table** containing rows & columns
- `Outer Query`: Selects from, filter, join and aggregate the **Inner Subquery**
	- `SELECT` lists the values produced by the subquery
```sql
SELECT t.country, t.total_arrivals 
FROM ( 
	SELECT country, SUM(arrivals) AS total_arrivals 
	FROM visitor_arrivals 
	GROUP BY country 
) AS t -- Alias for the inner subquery (MUST)
WHERE t.total_arrivals > 100000;
```

