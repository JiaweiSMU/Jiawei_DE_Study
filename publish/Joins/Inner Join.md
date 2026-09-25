
> [!NOTE] 
> Inner Join combines two table SIDE BY SIDE, putting columns from both tables into one row. It only keeps rows where the join condition finds a match in both tables
- It tries to find matching pairs (join condition)
- If a row from the left table has no match in the right table, it's gone, the same goes in reverse

### Using WHERE and aggregation with JOINs
The JOIN happens first and builds one combined table. 
Everything after it works on that combined table. 
1. JOIN → combine the tables into one 
2. WHERE → filter rows of the combined table 
3. GROUP BY → group/aggregate the filtered combined table 

So WHERE and GROUP BY see the joined result, not the original tables separately. You can use columns from either table in them.
### Quirk: 
NULL values in join columns cause row loss, since `NULL = NULL` and `NULL = anything` return unknown rather than true, and joins only keep pairs where the condition is true - so rows with NULL join keys never match anything, including other NULLs.