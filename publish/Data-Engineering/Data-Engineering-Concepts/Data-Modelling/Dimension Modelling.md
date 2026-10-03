![[Pasted image 20261003233715.png]]The goal is to take raw data and ==transform it into `Fact` and `Dimension`== tables that represent the business.
### How to do dimensional modelling
1.  **Declare the grain**: one sentence stating ==what a single row of the `fact table` represents== (`e.g.`: one row per month per country). 
	- Choose the ==lowest grain== the source provides. Going up is a `SUM()` and `GROUP BY` (daily -> monthly). Going down is impossible, because the detail was never stored. 
	- One grain per `fact table`. Mixing monthly total rows with daily rows makes any `SUM()` double count. 
2. **Identify the dimensions**: the descriptions that are true for a row at that grain (who, what, where, when).
	- Each dimension is its own table with one row per member and a key the `fact table` joins on (`e.g.`: country).
	- Its columns are the attributes (`e.g.`: region and continent are attributes of country).
3. **Identify the facts**: the **numeric measurements at that grain**. The `fact table` holds only the dimension keys and these numbers.
	- It is ==long and narrow== (many rows, few columns).
	- Check whether each fact can be summed:
		- A count can be summed across every dimension.
		- A balance can be summed across countries, but not across time.
		- A ratio cannot be summed. Store its numerator and denominator.
### Benefits
- Don't need to perform complex joins to the `Dimension` tables, can be done via the `surrogate keys`
- `Dimension` table **can be re-used** with other fact tables