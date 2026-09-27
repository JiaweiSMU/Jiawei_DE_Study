### WHERE
Narrows the final results down to only rows that match a condition.
```sql
SELECT
    device_id,
    status
FROM devices
WHERE status = 'active'
```

### Types of Comparison Operators
1. `=` — exact match.
2. `!=` / `<>` — values that don't match.
```sql
SELECT
    device_id,
    status
FROM devices
WHERE status != 'active'
```
3. `>`, `<`, `>=`, `<=` — greater than, less than, greater/equal, less/equal.
```sql
SELECT
    user_id,
    total_spent
FROM transactions
WHERE total_spent > 500

SELECT
    product_id,
    stock
FROM inventory
WHERE stock <= 10
```
4. `IN` — checks whether a value is in a specified list.
	- Similar to chaining ORs
```sql
SELECT id, type
FROM items
WHERE type IN ('book', 'tool', 'toy')

SELECT customer_id, name
FROM customers
WHERE customer_id IN (101, 202, 303)
```
5. `NOT IN` — checks whether a value is absent from a specified list.
```sql
SELECT id, status
FROM orders
WHERE status NOT IN ('cancelled', 'refund')
```
6. `AND` — combines multiple conditions, all of which **must be true.**
```sql
SELECT user_id, country, status
FROM users
WHERE country = 'Singapore'
AND status = 'active'
AND last_login > '2025'
```
7. `NOT` - keeps the rows where condition is false and drop the rows where it's true
	- Usually used with these:![[Pasted image 20260926224849.png]]
```sql
-- customers NOT from Singapore 
SELECT * FROM customers 
WHERE NOT country = 'SG'; 
-- same as: WHERE country <> 'SG'
```
8. `OR` - Keeps a row if **at least one of the condition is true**
	- **Watch out:** SQL evaluates `AND` before `OR`, so leaving out brackets can give results you didn't intend. Always use brackets when you mix `AND` and `OR`.
```sql
-- Without brackets, SQL reads this as: 
	-- country = 'SG' OR (country = 'MY' AND status = 'active') 
	-- Result: ALL SG customers + only ACTIVE MY customers
WHERE country = 'SG' OR country = 'MY' AND status = 'active'

-- The correct way is
-- With brackets: ACTIVE customers from SG or MY
WHERE (country = 'SG' OR country = 'MY') AND status = 'active'
```
