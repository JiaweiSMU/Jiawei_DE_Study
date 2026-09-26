> [!NOTE]
> Keeps every row from both tables. If the rows has no match, it will fill in with NULLS on the missing side

e.g:
```sql
SELECT s.id AS source_id, t.id AS target_id,
       s.amount AS source_amt, t.amount AS target_amt
FROM source s
FULL OUTER JOIN target t ON t.id = s.id;
```
![[Pasted image 20260926125512.png]]

### Common Uses
1. Finding Unmatched records from both tables
	- Do a `FULL OUTER JOIN` on the key, then keep the rows where **either side's key is NULL**:
### Quirks
MySQL does not support `FULL OUTER JOIN`, the workaround is to **combine 2 LEFT JOINs** 
> [!NOTE] Scenario
> Ana has two orders for 50, Ben has none, and order 103 points to a customer who doesn't exist.
> **Correct outcome:**
> ![[Pasted image 20260926133035.png]]
1. Using UNION:
	- UNION removes rows that are identical in the selected columns. Since the SELECT list doesn't include the order ID, her two rows look identical, and UNION keeps only one. ![[Pasted image 20260926133015.png]]
	- To keep all genuine rows, include the unique keys from both tables (`c.id` and `o.id`)
	```sql
	-- UNION Removes Duplicate Rows
	
	-- Keeps every customer & matched customer rows
		SELECT
			c.id as customer_id, c.name, o.id as order_id
		FROM cust c
		LEFT JOIN orders o ON o.customer_id = c.id
		
		UNION
		
	-- Keeps every over & matched customer rows
		SELECT 
			c.id, c.name, o.id
		FROM orders o
		LEFT JOIN customers c ON c.id = o.customer_id
	```
1.  Using UNION ALL
	-  The first query contains every customer, plus the orders that match a customer. It **misses the orders with no customer**.
	- The second query uses WHERE c.id IS NULL to **keep only those unmatched orders**. Its *matched rows are dropped because the first query already has them*.
	```sql 
		SELECT
			c.id as customer_id, c.name, o.id as order_id
		FROM cust c
		LEFT JOIN orders o ON o.customer_id = c.id
		
		UNION ALL
		
		SELECT 
			c.id, c.name, o.id
		FROM orders o
		LEFT JOIN customers c ON c.id = o.customer_id
		-- The filter is to keep rows where no customer is found
		WHERE c.id IS NULL
	```




