Rolling Average is usually calculated using an `AVG` expression + `WINDOW FUNCTION` together

Syntax:
```sql
AVG(expression) OVER (
    [PARTITION BY partition_column_1]
    ORDER BY sort_column [ASC | DESC]
    [ROWS | RANGE] BETWEEN frame_start AND frame_end
) AS alias_name
```
Difference between `Rows & Range`
- **`ROWS`**: counts physical rows relative to the current row (e.g. 2 rows above). The window size is fixed.
- **`RANGE`**: includes every row whose `ORDER BY` value falls within an offset of the current row's value (e.g. date − 2 days to date). 
	- The number of rows varies, and rows tied on the `ORDER BY` value are always included together.
Options for `frame_start` & `frame_end`:
```
UNBOUNDED PRECEDING   ← the very first row
      ...
  2 PRECEDING         ← 2 steps back
  1 PRECEDING         ← 1 step back
  CURRENT ROW         ← you are here
  1 FOLLOWING         ← 1 step forward
  2 FOLLOWING         ← 2 steps forward
      ...
UNBOUNDED FOLLOWING   ← the very last row
```

> [!NOTE] Rmb
> |You want|Frame|
|---|---|
|Running total (everything so far)|`BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`|
|Rolling 3 (me + 2 before)|`BETWEEN 2 PRECEDING AND CURRENT ROW`|
|Centred window (1 before, me, 1 after)|`BETWEEN 1 PRECEDING AND 1 FOLLOWING`|
|Whole partition|`BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING`|
