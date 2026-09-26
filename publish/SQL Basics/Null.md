### Quirks of NULL
> [!Example] 
>  A survey asking for income. NULL means the person skipped the question, `0` means they entered `0`, and an empty string is usually an input error.
1. NULL means ==unknown / not provided==.
2. `0` is not the same as NULL - `0` is a known quantity.
3. An empty string (`''`) is different from NULL too.
4. NULL isn't equal to another NULL - `NULL = NULL` doesn't return TRUE, it returns NULL.
### Comparisons using NULL
Any comparison involving NULL returns unknown (NULL), with zero exceptions. The only way to test for NULL is with ==`IS NULL` / `IS NOT NULL`==.
```sql
NULL = NULL  -- Returns NULL (not true!)
NULL <> NULL -- Returns NULL (not true!)
NULL > 5     -- Returns NULL
NULL = 'text'-- Returns NULL
```

### Handling NULL Values with ==COALESCE==
`COALESCE` replaces NULL with a value you specify.
```sql
SELECT
    cust_id,
    name,
    COALESCE(phone, 0) -- if NULL, sets phone to 0
FROM customers
```
### Count behaviours with NULL
- `COUNT(*)` counts all rows, including ones with NULL values. 
- `COUNT(column)` only counts rows where that column actually has a value, skipping NULLs. 
So with 10 rows where 3 have NULL in `discount`, `COUNT(*)` gives you 10, but `COUNT(discount)` gives you 7.

To find how many NULLs are in a column, subtract the two:
```sql
SELECT
    COUNT(*) - COUNT(col) as missing_vals
FROM table
```