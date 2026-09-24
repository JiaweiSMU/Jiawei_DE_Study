> [!NOTE] Order
> 1. `WHERE` runs before window function, so to filter the result of a window function, we need to wrap it as a CTE / Subquery and filter it.
> 2. `GROUP BY` runs before window functions, so aggregates are already computed by the time the window function runs — which means you can rank, partition, or otherwise window over an aggregate as if it were an ordinary column
> 	- The window function also sees the full grouped result of the `Group By` (e.g. one row per group)
### Filter Order
`WHERE` runs before window function, so to filter the result of a window function, we need to wrap it as a CTE / Subquery and filter it.
Order:
1. **`WHERE` in the inner query — runs first, before the window function.** This decides which rows the window function is even allowed to see. 
2. **The window function runs** on whatever survived step 1.
3. **`WHERE` in the outer query — runs last.** This filters on the window function's output, which is why you need the CTE. `rn = 1` goes here.
### Syntax
```sql
-- Function: 
	--Aggregates (e.g. SUM, AVG)
	-- Ranking (e.g. ROW_NUMBER, RANK() )
	-- Offset (e.g. LAG(), LEAD() )
<function>(<args>) OVER ( 
	[PARTITION BY col1, col2, ...] 
	[ORDER BY col3 [ASC|DESC] [NULLS FIRST|LAST]] 
	[ROWS | RANGE BETWEEN <start> AND <end>] -- Rows / Range
)
-- Rows / Range:
	-- Unbounded Preceding: First row of the partition
	-- n Preceding: n rows before current
	-- Current Row: This row
	-- n Following: n rows after current
	-- Unbounded Following: Last row of the partition
```
### Difference between Group By & Window Function
Both do calculations over a set of related rows. The difference is what comes back.
- `GROUP BY`: Collapse rows into groups and return **ONE ROW PER GROUP**
- `WINDOW FUNCTION`: Perform calculations across a **WINDOW** of rows **without collapsing anything**, with the grouped answer added as an extra column.
`Window`: The set of rows the function is allowed to look at when computing the result for the current row. The `OVER (...)` clause is what defines that set.
### Defining the window
- `OVER ()` with empty parentheses means the window is _all rows_ in the result, so every row gets the same value.
- `OVER (PARTITION BY col)` splits rows into ==groups by== that column — the function restarts for each group, so a row only sees rows sharing its value.
```sql
-- Average salary across the whole table; same number on every row
SELECT name, salary,
       AVG(salary) OVER () AS overall_avg
FROM employees;

-- Average salary within each department; Sales rows get Sales' average
SELECT name, dept, salary,
       AVG(salary) OVER (PARTITION BY dept) AS dept_avg
FROM employees;
```
So `PARTITION BY dept` produces the same numbers as `GROUP BY dept` — it just keeps the rows instead of collapsing them.
## Ranking Functions:
1. `ROW_NUMBER()`: Gives a unique number for every row even for ties (e.g. 1, 2, 3, 4)
2. `RANK()`: Gives same number for ties, ==**SKIPS** number== (e.g. 1, 1, 3, 4, 4, 6)
3. `DENSE_RANK()`: Gives same number for ties, ==**NO SKIP** of number== (e.g. 1, 1, 2, 3, 3, 4)
![[Pasted image 20260921205313.png]]
### ROW_NUMBER()
`ROW_NUMBER()` gives a number (1, 2, 3) to the rows in each partition, and the ==`ORDER BY`== inside the `OVER()` decides **who gets to be number 1**
- Common use is **deduplication**: keeping only the most recent (or highest, or first) version of each record.
```sql
-- Groups rows by employee id, sorts each group highest salary first, then keeps only the top row per employee
WITH x AS (
    SELECT id, name, dept_id, salary,
           ROW_NUMBER() OVER (PARTITION BY id ORDER BY salary DESC) AS rn
    FROM employees
)
SELECT * FROM x WHERE rn = 1;
```
