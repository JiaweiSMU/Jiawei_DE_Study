### Group By
Collapses individual rows with the same value into the same bucket, then calculates something for that bucket.  ![[Pasted image 20260914215126.png]]
```sql
SELECT
    region,
    COUNT(*) AS sale_count
FROM transactions
GROUP BY region
```
When **grouping by more than one column, each unique combination of values becomes its own group**.
```sql
SELECT
    region,
    product,
    SUM(amount) AS total
GROUP BY region, product
```

**Before**: 
![[Pasted image 20260914215537.png]]
**After**:  ![[Pasted image 20260914215516.png]]
#### Rules of Group By
1. Every column in the SELECT must either be in the GROUP BY clause or wrapped in an aggregate function (COUNT, SUM, AVG, etc.)
### Types of Aggregations
#### Counts
1. `COUNT(*)` — how many rows are in each group.
2. `COUNT(<col>)` — counts only rows where that specific column **is not NULL.**
3. `COUNT(DISTINCT <col>)` — counts only **unique values, ignoring duplicates.**
#### SUM
Gets totals broken down by category (whatever you grouped by).
![[Pasted image 20260914220159.png]]

**Quirks**:
1. NULL values are skipped
2. Only work on NUMERIC columns
3. Returns NULL if all values are NULL
#### AVG
Calculates the mean (sum of values divided by count of values) for each group.

**Quirk**: NULLs are skipped from both the sum and the count - so a group with values `10, NULL, 20` averages to `15` (30/2), not `10` (30/3).
#### MIN & MAX
`MIN` gets the smallest value per group, `MAX` gets the largest. Both work beyond numbers too - on dates, and on text using alphabetical ordering.
```sql
SELECT
    cust_id,
    MIN(order_date),
    MAX(order_date)
FROM orders
GROUP BY cust_id
```
#### HAVING
**Filters** groups **after aggregation**, the same way WHERE filters rows before aggregation. 
The reason you need a separate keyword is that WHERE runs before grouping even happens, so it has no access to aggregate values like `COUNT(*)` or `SUM(amount)` to filter on.

You can use both in the same query - when you do, they run in this order: 
- WHERE filters individual rows first, GROUP BY collapses what's left into groups, then HAVING filters those groups based on the aggregate.
```sql
SELECT
    region,
    COUNT(*) AS completed_orders
FROM orders
WHERE status = 'completed'
GROUP BY region
HAVING COUNT(*) > 2
```