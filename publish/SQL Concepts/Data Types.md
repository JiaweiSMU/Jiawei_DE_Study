There are three main categories:
1. `INTEGER`
2. `VARCHAR` (text strings)
3. `BOOLEAN` (true/false)
### Casting
Used to convert a value from one type to another - for example, when `'5'` is stored as a `VARCHAR` and you want it as an `INTEGER`. 
Casting fails if the conversion isn't possible, like trying to cast `'abc'` as an integer.
```sql
SELECT
    CAST('5' AS INTEGER) as new_int
FROM users
```
