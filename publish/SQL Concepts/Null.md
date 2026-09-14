### Quirks of NULL

> [!Example] 
> A survey asking for income. NULL means the person skipped the question, 0 means they selected 0 and empty string is usually a input error
1. NULL refers to Unknown / Not provided
2. 0 is not the same thing as NULL, **0 is a KNOWN QUANTITY**
3. An empty string (`''`) is different from NULL.
4. NULL is not equal to another NULL
	1. ==NULL = NULL doesn't return TRUE==, it returns NULL
### Comparisons using NULL
Any comparison involving NULL returns unknown (NULL), with zero exceptions.
The only way to test for NULL is by using `IS NULL` / `IS NOT NULL`

### IS NULL / IS NOT NULL
Since `=` or `!=` can't be used to check for NULL. We need to use `IS NULL` or `IS NOT NULL`

```
Select
	customer_id,
	name,
	phone
from customers
WHERE phone IS NULL

Select
	customer_id,
	name,
	phone
from customers
WHERE phone IS NOT NULL
```

### Handling NULL Values with ==COALESCE==
Coalesce replaces NULL with a value
```
SELECT
	cust_id,
	name,
	COALESCE(phone, 0) # If null, sets phone number as 0
from customers
```

### Count behaviours with NULL
> [!Example] 
> So if you've got 10 rows and 3 of them have NULL in the discount column, COUNT(*) gives you 10, but COUNT(discount) gives you 7
- COUNT(`*`) will count all rows including those with NULL values
- COUNT(column) only counts rows where that specific column actually has a value, skipping any row where it's NULL

If we want to find out how many NULL values are in a particular column
```
SELECT
	COUNT(*) - COUNT(col) as missing_vals
from table
```
