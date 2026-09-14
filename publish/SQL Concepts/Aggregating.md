### Group By
This collapses individual rows with the same value into the same bucket then calculate something for that bucket![[Pasted image 20260914215126.png]]
```
SELECT
	region,
	count(*) AS sale_count
FROM transactions
GROUP BY region
```

When grouping by more than 1 column, each unique combination of values becomes its own group
```
SELECT 
	region,
	product,
	sum(amount) AS total
group by region, product
```

Before:
![[Pasted image 20260914215537.png]]

After:
![[Pasted image 20260914215516.png]]
#### Rules of Group By
1. Every column in the SELECT must be in the GROUP BY clause
2. Inside an aggregate function (COUNT, SUM, AVG etc.)

## Type of Aggregations
### Type of Counts
1. COUNT(`*`) - tells you how many rows are in each group
2. COUNT(<COL_NAME>) - Counts only rows where that specific column is **NOT EMPTY / NULL**
3. ==COUNT(DISTINCT)== - Only counts for **UNIQUE values**, ignoring duplicates
### Sum
Get totals broken down by category (What's being grouped by)![[Pasted image 20260914220159.png]]
Quirks:
1. NULL values are skipped
2. Only work on NUMERIC columns
3. Returns NULL if all values are NULL
### AVG
Calculates the **mean** (Sum of all values divided by count of values) for each group

Quirks
1. NULLS are skipped (e.g. if a group has values `10, NULL, 20`, the average is `15` (30/2), not `10` (30/3))
### MIN & MAX
MIN - Gets the smallest value for each group
MAX - Gets the largest value for each group

Able to use it beyond numbers. Works for:
- **dates, text** (alphabetical ordering)
```
SELECT
	cust_id,
	MIN(order_date),
	MAX(order_date)
from orders
group by cust_id
```
### HAVING
HAVING filters groups **after aggregation**, the same way WHERE filters rows before aggregation
- The reason you need a separate keyword is that WHERE runs before grouping even happens, so it has no access to aggregate values like `COUNT(*)` or `SUM(amount)` to filter on.

You can use both in the same query, and when you do, they run in this order: WHERE filters individual rows first, then GROUP BY collapses what's left into groups, then HAVING filters those groups based on the aggregate.
```
SELECT
	region,
	count(*) as completed_orders
FROM orders
where status ='completed'
GROUP BY region
HAVING COUNT(*) >2
```
