A self join joins a table to itself. It's used when rows in a table relate to other rows in the same table. Giving the table two different aliases lets you treat it as two separate copies, so you can match one row against another.
```sql
SELECT
  e.name AS employee,
  m.name AS manager
FROM employees e
LEFT JOIN employees m
  ON e.manager_id = m.id;
```
![[Pasted image 20260926223416.png]]

