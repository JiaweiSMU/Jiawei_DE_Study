### Type of Pattern Matching
1. `LIKE` - Search for patterns in text using wildcard characters
	- `%`: stands for **any number of characters**, including none.
	- `_`: stands for **exactly one character**, no more and no fewer.
```sql
WHERE name LIKE 'Jo%' -- starts with "Jo": Jo, John, Joanna 
WHERE name LIKE '%son' -- ends with "son": Jason, Allison 
WHERE name LIKE '%an%' -- contains "an" anywhere: Dan, Anna, Joanna

WHERE code LIKE 'A_1' -- A11, AB1, AX1 (but not A1 or AB21) 
WHERE name LIKE '___' -- any name exactly 3 characters long
```
2. `BETWEEN`: Used to filter for values within a range. It's **INCLUSIVE** (Includes the two values specified)
```sql
WHERE age BETWEEN 18 AND 30 
-- keeps 18, 19, 20 ... 30 
-- same as: age >= 18 AND age <= 30
```
