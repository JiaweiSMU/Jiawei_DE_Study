### `CONCAT()` & `CONCAT_WS()`
![[Pasted image 20260927205653.png]]

| Situation                       | Use                             |
| ------------------------------- | ------------------------------- |
| Joining text, no NULLs possible | `CONCAT(a, ' ', b)`             |
| Some pieces might be NULL       | `CONCAT_WS(' ', a, b, c)`       |
| Want a placeholder for NULL     | `CONCAT(a, COALESCE(b, 'N/A'))` |
#### CONCAT()
Join different strings into **ONE string**
```sql
SELECT CONCAT(first_name, ' ', last_name) AS full_name FROM employee; 
-- 'Max', ' ', 'George' becomes 'Max George'
```
- Can be used on 'non-string' values as well
- If any of the value being passed to `CONCAT()` is `NULL`, the ==whole result will be `NULL`==.
	- Either use `CONCAT_WS()` or use `COALESCE()`
#### CONCAT_WS()
Joins the values with the separator placed between each one. Any `NULL` values are skipped, and the remaining values are joined together.
```sql
CONCAT_WS(' ', first_name, middle_name, last_name) 
		-- ↑ separator       ↑ the pieces to join
```
- Separator can be anything
### Case Conversion `UPPER() & LOWER()`
`LOWER()` converts letters to lowercase, `UPPER()` converts them to uppercase
- non-letter characters are left unchanged either way. 
- Both only transform the output; the underlying data isn't modified.
### Removing Whitespace `TRIM()`
It's able to remove trailing spaces
```sql
-- Remove leading/trailing spaces
SELECT TRIM('  hello world  ');
-- 'hello world'

-- Clean messy data before comparing
SELECT
  TRIM(product_name) AS clean_name
FROM products
WHERE TRIM(product_name) = 'Widget';
```