Syntax:
```sql
WITH cte_name AS (
  SELECT ...
  FROM ...
  WHERE ...
),
second_cte AS (
	SELECT ...
	FROM...
	WHERE...
)
SELECT *
FROM cte_name;
```
### Pre-Aggregate before Joining
> [!NOTE] Sanity check
> **Can a row from the table whose column I'm summing match more than one row on the other side?** If yes, that value will be repeated, so pre-aggregate the other side first.

When doing a **many-to-one join**, the number of rows doesn't multiply, each row from the "one" side is repeated for every matching row on the "many" side. It's safe to aggregate columns from the "many" side after the join without pre-aggregating. ==Aggregating columns from the "one" side after the join gives the wrong number==, because those values are repeated.

When doing a many-to-many join, rows from both sides are repeated, so the number of rows multiplies. Aggregating columns from ==either side after the join gives the wrong number==. 
Hence, there is a need to pre-aggregate before joining.
**For Example**:
![[Pasted image 20260927114622.png]]
#### Many to One
```sql
-- Many employee rows point to 1 department
SELECT d.department, SUM(e.salary) AS total_salary
FROM employees e
JOIN departments d ON e.department = d.department
GROUP BY d.department;
```
![[Pasted image 20260927115035.png]]
- Aggregating `MANY` side (Salary) is safe, but if aggregating `One` side (Budget), we will get the wrong value
#### Many to Many
```sql
SELECT e.department,
       SUM(e.salary) AS total_salary,
       SUM(p.cost)   AS total_cost,
       COUNT(*)      AS headcount
FROM employees e
JOIN projects p ON e.department = p.department
GROUP BY e.department;
```
![[Pasted image 20260927115316.png]]
Instead it's better to Pre-Aggregate into one row per key before joining:
```sql
-- Get total salary for department first
WITH emp AS ( 
	SELECT department, SUM(salary) AS total_salary, COUNT(*) AS headcount 
	FROM employees 
	GROUP BY department 
), 
-- Get total cost for project
proj AS ( 
	SELECT department, SUM(cost) AS total_cost 
	FROM projects 
	GROUP BY department 
) 
SELECT 
	d.department, d.budget, emp.total_salary, emp.headcount, proj.total_cost 
FROM departments d 
	LEFT JOIN emp ON emp.department = d.department 
	LEFT JOIN proj ON proj.department = d.department;
```