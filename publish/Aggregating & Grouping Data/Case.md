It's SQL way of performing `If-Then-Else`. It checks condition one by one and returns a value for the **first** one that's true.

Syntax:
```sql
SELECT name,
       age,
       CASE
         WHEN age < 18 THEN 'Minor'
         WHEN age < 65 THEN 'Adult'
         ELSE 'Senior'
       END AS age_group
FROM customers;
```

### Using Case with Aggregation
> [!NOTE] How it works
> For each row, `CASE WHEN` checks the condition. If true, it returns 1. If false, 0. `SUM` adds up all those 1s and 0s.
1. Using `COUNT`
```sql
SELECT
	COUNT(CASE WHEN status = 'active' THEN 1 END) AS active_count,
    COUNT(CASE WHEN status = 'inactive' THEN 1 END) AS inactive_count
FROM customers;
```
2. Using `SUM`
```sql
SELECT
	SUM(CASE WHEN department = 'HR' THEN 1 ELSE 0 END) AS hr_count,
	SUM(CASE WHEN department = 'Admin' THEN 1 ELSE 0 END) AS admin_count,
	SUM(CASE WHEN department = 'Account' THEN 1 ELSE 0 END) AS account_count
FROM techcorp_workforce;
```