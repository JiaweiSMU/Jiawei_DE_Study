```
SQL Query Execution Logic
1. FROM
2. JOIN
3. WHERE
4. SELECT
```
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

To cont: https://www.stratascratch.com/learn/comprehensive-sql/partition-by-and-ranking-functions
### RANK
There are 3 different types
1. ROW_NUMBER - Gives every row a unique number
2. RANK - Possible to have ties e.g. (1, 1, 3). If so, it jumps a rank
3. DENSE_RANK - Ties share the same rank, there is no jump
	1. Whenever question ask if **TIED show all**, use DENSE_RANK

### LAG & LEAD
These are a form of **WINDOW FUNCTION** and need to be used together with <u>*ORDER BY inside OVER(...)*</u>
> Syntax for LAG / LEAD
> 	LAG(column, offset, default)
> 	Offset refers to get value from X row
- LAG - Gets value from the row before
- LEAD - Get value from a later row