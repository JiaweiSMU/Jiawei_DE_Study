Returns every row from the first table paired with every row from the second. There's no join condition. If table A has _m_ rows and table B has _n_ rows, the result has _m × n_ rows.
```sql
-- e.g.

-- months: Jan, Feb, Mar (3 rows)
-- markets: SG, MY (2 rows)

SELECT m.month, k.market
FROM months m
CROSS JOIN markets k
```
![[Pasted image 20260926222616.png]]
