### String Literals
A string literal returns the same exact text for every row.
```sql
-- For every row, col_a returns the value 'A'
SELECT
    'A' as col_a,
    col_b
FROM table
```
### String Functions
#### CONCAT()
Joins together multiple strings into a **single** string by placing them **end to end** in the order specified.
```sql
SELECT
    CONCAT(first_name, ' ', last_name) as name
FROM users

SELECT
    CONCAT(UPPER(region), '-', order_id) AS region_order_id
FROM orders
```
- It converts non-string arguments to strings before joining - `CONCAT('Total: ', 100)` gives you `'Total: 100'`.
#### LENGTH()
Returns the number of characters in a string, counting letters, numbers, spaces, and punctuation alike.
```sql
SELECT
    username,
    password
FROM users
WHERE LENGTH(password) < 8
```
#### LOWER() / UPPER()
`LOWER()` converts letters to lowercase, `UPPER()` converts them to uppercase
- non-letter characters are left unchanged either way. 
- Both only transform the output; the underlying data isn't modified.
#### Quirks of Strings
1. String comparisons are case-sensitive.