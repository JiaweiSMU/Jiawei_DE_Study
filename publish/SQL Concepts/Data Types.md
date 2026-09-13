There main categories:
1. INTEGER
2. VARCHAR (Text Strings)
3. BOOLEAN (True / False)

### Casting
Used to convert values from one type to another.
- e.g: When '5' is stored as a VARCHAR and we want to convert it to INTEGER
Will fail to cast if conversion is not possible (e.g. CASTING 'abc' as INTEGER)

```
SELECT
	CAST('5' AS INTEGER) as new_int
from users
```