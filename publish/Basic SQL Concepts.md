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
### SQL Basics
- [[Filtering]]
- [[Pattern Matching]]: Filtering but using LIKE (`%` & `_`) and BETWEEN
- [[Null]]: Quirks of NULLs, filter for NULLs & how to rectify if a column has NULLs using COALESCE
- [[Query Structure]]: Using Expressions & Aliases
### Aggregating & Grouping Data
- [[Grouping, Aggregate Functions & Having]]
- [[Case]]
### Joins
NULL values in join columns cause row loss, since `NULL = NULL` and `NULL = anything` return unknown rather than true, and joins only keep pairs where the condition is true - so rows with NULL join keys never match anything, including other NULLs.
> [!Cheatsheet] Cheatsheet
> The deciding question for all of them is the same: **when a row has no match, should it appear in the result, and from which side?**
> 
> |Join|Keeps|Unmatched rows|Typical use|
|---|---|---|---|
|**INNER**|Only matches|Dropped from both sides|Records that must exist in both tables|
|**LEFT**|All left rows|Right columns become NULL|Main list plus optional details; anti-join (`IS NULL`) to find missing|
|**RIGHT**|All right rows|Left columns become NULL|Rarely; rewrite as LEFT|
|**FULL OUTER**|Everything|NULLs on whichever side is missing|Reconciliation, finding gaps on both sides|
|**CROSS**|Every combination|No matching involved|Building grids (dates × categories)|
|**SELF**|Depends on join used|Depends on join used|Comparing rows in the same table (employee vs manager)|
- [[Union]]: Stack results of 2 queries 
- [[Inner Join]]: Keep only rows that have a match in both tables.
- [[Left Join]]: Keep every row from the left table. Where the right table has a match, we get its columns; otherwise they're NULL.
- [[Full Outer Join]]: Keep every row from both tables. Matched rows are combined, and unmatched rows get NULL for the missing side's columns.
- [[Cross Join]]: Pairs **every row in one table with every row in the other**. Resulting in (m * x) num of rows
- [[Self Joins]]: Used when rows in a table relate to oneself (E.g. Employee table containing employees and manager)
### Subqueries & CTEs
- [[Subqueries]]: Various type of subqueries (Filtering, Select & From)
- [[CTEs]]: When to Pre-Aggregate in a CTE before joining
- [[CTE Patterns for Common Problems]]
### Date, Time & Text Functions
- [[Time Data Types & Extraction]]: Various Date / Time Data type & How to extract parts of date
- [[Date Arithmetic & Truncation]]
- [[String Function]]
### [[Window Function]]
- [[Ranking]]
- [[Lead & Lag]]: Look forward / backwards

DISTINCT
- When using DISTINCT itself, it applies to every column in the SELECT list
