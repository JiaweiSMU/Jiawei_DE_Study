### VLookUp
**What it does:** Searches for a value in the first column of a range, then returns a value from another column in the same row. Think of it as a LEFT JOIN that brings back one column.
```
=VLOOKUP(lookup_value, table_array, col_index_num, range_lookup)
```

| Argument        | In plain words                                                                                                                                                                                                                                | Example                           |
| --------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------- |
| `lookup_value`  | `Key` you are searching for                                                                                                                                                                                                                   | `C2` (the CustomerID in this row) |
| `table_array`   | **Start at the column that holds the key** and stretch right to the column you want back<br><br>e.g. If the key is in column B then it should start from column B. To return the value in column D, then the range need to extend to column D | `Customers!$B:$D`                 |
| `col_index_num` | **Counted from the start of the range** (first column = 1)                                                                                                                                                                                    | `2` → Segment                     |
| `range_lookup`  | `FALSE` = exact match (the default choice). `TRUE` = approximate match (only for bands, and the table must be sorted)                                                                                                                         | `FALSE`                           |
**Read it out loud**
`=VLOOKUP(C2, Customers!$B:$D, 2, FALSE)`
→ Find C2 in column B of Customers, move 2 columns along, and return that value. Exact matches only.

**Counting columns**
![[Pasted image 20261013230550.png]]

**Lock the range with `$`** so it doesn't shift when you fill down
```
Customers!$B$2:$D$41   ← fixed range, rows 2 to 41
Customers!$B:$D        ← whole columns (quicker, picks up new rows)
```
### XLOOKUP
A better version of VLOOKUP: it can **return from any column (left or right)** and defaults to exact match.
```
=XLOOKUP(lookup_value, lookup_array, return_array, [if_not_found], [match_mode], [search_mode])
```
- `lookup_array` = the **one column** to search
- `return_array` = the column(s) to bring back (must be the same height as `lookup_array`)
- `if_not_found` = what to show instead of `#N/A`

```
-- Search column B for the key, return the value from column A
=XLOOKUP(C2, Customers!$B:$B, Customers!$A:$A, "Not found")
```
test