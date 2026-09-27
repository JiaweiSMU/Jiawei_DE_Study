### Safe Ratio & Percentage
Percentages can break in 2 ways:
#### 1. Dividing by 0 → the query errors out
If any row has a denominator of 0, the whole query fails.

**Fix:** wrap the denominator in `NULLIF(denominator, 0)`, which means "if the ==denominator is 0, use NULL instead==." Anything divided by NULL is NULL, so that row returns a blank instead of crashing the query.
```sql
-- Crashes if denominator is 0
SELECT numerator / denominator AS ratio;

-- Returns NULL for that row instead
SELECT numerator / NULLIF(denominator, 0) AS ratio;
```
#### 2. Integer division → decimals get thrown away
When both numbers are whole numbers (integers), the result is also a whole number, and anything after the decimal point is cut off.

| Calculation   | You expect | You get                      |
| ------------- | ---------- | ---------------------------- |
| `3 / 4`       | 0.75       | **0**                        |
| `3 * 100 / 4` | 75         | 75 (works here only by luck) |
| `1 * 100 / 3` | 33.33      | **33**                       |
**Fix:** make one of the numbers a decimal. If any number in the calculation is a decimal, the result is a decimal. For percentages, the easiest way is to write `100.0` instead of `100`:
```sql
count_a * 100.0 / NULLIF(count_total, 0) AS pct
```
- `100.0` makes the calculation decimal, so `1 * 100.0 / 3` gives `33.33`
- `NULLIF` protects against dividing by 0
### Comparing rows within the same table
Given that we need to find employees who earn more than their managers, and managers are also rows in the `employee` table, linked by `manager_id`.
1. Use a CTE to create a lookup of each person's `id` and salary, renamed to `manager_salary`.
2. Join the employee table to the CTE, matching the employee's `manager_id` to the CTE's `id`. Each employee row then has their manager's salary alongside it, so the two can be compared.
```sql
WITH manager_salaries AS ( 
	SELECT 
		id, salary AS manager_salary 
	FROM employee 
) 
SELECT e.first_name, e.salary 
FROM 
	employee e JOIN manager_salaries m 
	ON e.manager_id = m.id 
WHERE e.salary > m.manager_salary;
```