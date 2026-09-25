A join pairs up rows from two tables by testing each row in one table against each row in the other, keeping only the pairs that match some condition. 
- SQL produces one output row per matched pair.
![[Pasted image 20260915220606.png]]
### Types of Joins
### Quirk: 
NULL values in join columns cause row loss, since `NULL = NULL` and `NULL = anything` return unknown rather than true, and joins only keep pairs where the condition is true - so rows with NULL join keys never match anything, including other NULLs.
