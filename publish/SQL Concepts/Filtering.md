### WHERE
Narrow down the final results to only rows that match a condition
```
Select
	device_id,
	status
FROM devices
WHERE status = 'active'
```

### Types of Comparison Operators
1. '=': To check for EXACT match
2. '**!= / <>'**: To check where values DON'T match
	```
	   SELECT
		   device_id,
		   status
		FROM devices
		WHERE status != 'active'
	```
3. '>, <, >=, <=': For comparing (More than, Less than, More/Equal, Less/Equal)
	```
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
4. 'IN': To check values are IN the list of specified things
	```
	SELECT id, type
	FROM items
	where type IN
	('book', 'tool', 'toy')
	
	SELECT customer_id, name
	FROM customers
	WHERE customer_id IN
	(101, 202, 303)
	```
5. 'NOT IN': To check values are NOT IN the list of specified things
	```
	SELECT id, status
	FROM orders
	WHERE status NOT IN
	('cancelled', 'refund')
	```
6. 'AND': To combine multiple rules
	```
	SELECT user_id, country, status
	FROM users
	WHERE country ='Singapore'
	AND status ='active'
	AND last_login > '2025'
	```


