
> [!NOTE] 
> Keeps EVERY row from the Left table, if the right side has a match, the column will be added else the column will show up as NULL.
e.g.:
![[Pasted image 20260926115429.png]]
### Common uses
1. Return rows in table A that has **NO MATCH** in table B.
	- LEFT JOIN + IS NULL (On right table `PK`)
### LEFT JOIN: ON vs WHERE
- **Right-table filter → ON** (`ON ... AND t.country = 'JP'`). It decides what counts as a match, and unmatched left rows are kept. In WHERE, it would drop them, because a comparison with NULL is never true.
- **Left-table filter → WHERE** (`WHERE v.name <> 'Ben'`). It removes left rows. In ON, those rows would still appear, with NULLs.