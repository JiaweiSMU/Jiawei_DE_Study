Column cardinality is the number of **distinct values in a column**: a ==column is unique only when its cardinality equals the table's row count==.
- e.g. Month, if row count > 12 than the column is not unique.

High and low are judged by how many rows each value covers:
- **High cardinality**: many distinct values, so each value matches a small fraction of the rows.
	- e.g. `user_id` with 10 million users. The column repeats values, so it is not unique, but it is still high cardinality. In a `users` table, the same column is unique.
- **Low cardinality**: few distinct values, so each value matches a large fraction of the rows.
	- e.g. a boolean column (2 values), `region` (5 values).

