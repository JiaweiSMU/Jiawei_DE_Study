### Histogram Pattern
A histogram maps **values → frequency of occurrence**. 
- Count **per item occurrence** first (e.g. user posted how many tweets), then **count how many items share the same count.**
	1. Group by item, then count
	2. Group by the count of the item
### Relational Division Problem
Such questions are usually in this format - "*Find all X that are associated with all of a given set of Y*"
- Example questions include
	- Find **customers** who bought **all** products in a bundle
	- Find **students** who passed **all** required courses
	- Find **employees** who completed **all** mandatory trainings
	- Find **suppliers** who stock **all** parts needed for a product
- Usually the table will have a Identifier column (e.g. User) and things that are associated with the user. There are usually duplicate identifier within the table

To solve such a question
1. Making use of **IN**, **GROUP BY** & **HAVING + COUNT**
	1. skills in ('stuff we are searching for') -> Group by Identifier -> Having count() - Depends on how many they must match
2. Using **SELF-JOINS** 
	1. select a.id from table a 
		   join table b on a.id = b.id and skill = 2
		   join table c on a.id = c.id and skill = 3
		where a.skill = 1 (**This must be at the end can't put at the start**)

### Anti Join
These type of questions usually ask, "*Find records with no match in another table*"

Two ways to solve this problem
1. LEFT JOIN + IS NULL
	1. Join table A and B, check that the key in table B is NULL 
2. SUBQUERY with NOT IN
	1. Select query where id not in (select id from table B)

### Conditional Aggregation Pattern
Where *Aggregate* (SUM, COUNT, AVG) functions are selected based on some kind of condition
- To take note, always **END** before the closing ')'
- The conditional arg is not using 'WHERE' but '**WHEN**'
> Don't overthink the CASE operations, if it's a SUM then just call the Col_value, else just do a -Col_value

Types
1. SUM(CASE when col_name = 'A' then 1 else 0 end) as new_name
2. SUM(CASE when col_name in ('A', 'B') then 1 else 0 end) as new_name

### Finding Duplicates Pattern
There are 3 types 
1. Identifying duplicates group, to count how many duplicates there are ![[Pasted image 20260503151912.png]]
	1. Done using **Group By**
2. Finding out which pairs are duplicated - To see which rows match which rows
	1. Done using **Self Join**
3. For labelling duplicates then either keep one or mark the others![[Pasted image 20260503152020.png]]
	1. Done using **Row Number**

### Rolling average for next 7 rows
Use an **AVG aggregate** together with **Window Function** 

Code:
> Select
> 	id, amount,
> 	AVG(amount) over(order by id 
> 	<u>ROWS BETWEEN CURRENT ROW and 6 FOLLOWING</u>) 
> as rolling_avg_next_7
Types:
- Look forward
	- Following - Looks at rows after current row
		- ROWS BETWEEN CURRENT ROW and 6 FOLLOWING 
- Look backwards
	- Preceding - Looks at rows before current row
		- ROWS BETWEEN 6 PRECEDING AND CURRENT ROW

