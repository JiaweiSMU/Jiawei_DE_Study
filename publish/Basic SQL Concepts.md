## SQL Query Execution Logic
1. **`FROM`** — builds the working set of rows from the base table.
2. **`JOIN`** — combines tables. A condition in `ON` filters what counts as a match; the same condition in `WHERE` runs later and drops NULL-padded rows, turning a `LEFT JOIN` into an inner join.
3. **`WHERE`** — filters individual rows. Can't see aliases, aggregates, or window functions — none of them exist yet.
4. **`GROUP BY`** — collapses rows into groups. **Aggregates are computed here**, and the raw rows are gone afterwards.
5. **`HAVING`** — filters the grouped results. This is where aggregate conditions go, e.g. `HAVING SUM(n_messages) > 100`, since `WHERE` ran too early to see them.
6. **Window functions** — run over the grouped rows. Why `SUM(n_messages)` works inside `OVER()` but a bare `n_messages` doesn't.
7. **`SELECT`** — produces the output columns. **Aliases only exist from here on.**
8. **`DISTINCT`** — dedupes whatever `SELECT` produced.
9. **`ORDER BY`** — sorts the final result. Runs after `SELECT`, so it _can_ use aliases.
10. **`LIMIT` / `OFFSET`** — trims the sorted result.
### [[Query Structure]]
How to use AS and EXPRESSIONS when querying
### [[Data Types]]
The various Data Types & Uses of CASTING
### [[Filtering]]
Goes through the various operators and examples
### [[Aggregating]]
Goes through Group By, various Aggregations Functions & Quirks
### [[Null]]
The quirks of NULLs, how to filter for NULLs & how to rectify if a column has NULLs using COALESCE if we want it to have some form of value
### [[Joins]]
Inner joins and some quirks of joins
### [[String Functions]]
Going through some basic string functions (Concat, Lower, Upper & Length)
### [[Dates]]
The difference between DATE & Timestamp, usefulness of DATE_ADD() and DATE_DIFF() functions
### [[Window Function]]
- [[Ranking]]
- [[Lead & Lag]]


### LAG & LEAD
These are a form of **WINDOW FUNCTION** and need to be used together with <u>*ORDER BY inside OVER(...)*</u>
> Syntax for LAG / LEAD
> 	LAG(column, offset, default)
> 	Offset refers to get value from X row
- LAG - Gets value from the row before
- LEAD - Get value from a later row