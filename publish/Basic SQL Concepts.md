### [[Query Structure]]
How to use AS and EXPRESSIONS when querying
### [[Data Types]]
The various Data Types & Uses of CASTING
### [[Filtering]]
Goes through the various operators and examples
### [[Aggregating]]
Goes through Group By, various Aggregations Functions & Quirks
### [[Null]]


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