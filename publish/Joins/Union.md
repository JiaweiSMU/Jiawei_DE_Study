
> [!NOTE] 
> UNION stacks the results of two or more queries VERTICALLY (**Rows are APPENDED** leading to increase in num of rows)

**Syntax**: Results are stacked on top of one another
```sql
SELECT col1, col2
FROM table_a

UNION

SELECT col1, col2
FROM table_b
```

In `UNION`, both SELECTS must return the same number of columns & the data types must be compatible.
- If a table is missing a column, a placeholder can be used (e.g. NULL AS 'col')

### Types of Union 
1. UNION – Duplicates are removed 
	- Compares the WHOLE ROW (whichever columns are selected make up the whole row)
	- Removes duplicates within each query too (e.g. If `A` appears twice within table_a, it gets removed)
2. UNION ALL – Everything is kept

