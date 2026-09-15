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
6. `AND` — combines multiple conditions, all of which must be true.
```sql
SELECT user_id, country, status
FROM users
WHERE country = 'Singapore'
AND status = 'active'
AND last_login > '2025'
```
