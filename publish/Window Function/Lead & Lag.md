LAG and LEAD let a row look at another row's value — `LAG` looks backward, `LEAD` looks forward, and you can choose how many rows away.
```sql
LAG(expression , offset, default]])  OVER ([PARTITION BY col] ORDER BY col)
LEAD(expression , offset, default]]) OVER ([PARTITION BY col] ORDER BY col)
```
- **Expression**: ==**Column Name / Expression**== from the Lagged / Lead row![[Pasted image 20260922211834.png]]
	- It can do **aggregations** provided that there's a GROUP BY
- **Offset**: How many rows away to look, **default is 1**. It ==counts rows==, not calendar time, so `LAG(visitors, 12)` is only "a year ago" if the series has no missing months.
- **Default**: Value to return when there's no row at that offset, if left empty it just returns 0
- **Order** **By**: Defines what "before" and "after" mean. Rows have no inherent order, so without it there's nothing to step backwards / forward
- **Partition By**: 

> [!NOTE] Common uses
> 1. To get the time since someone's last action
> 2. Period over Period change
### Ways to handle NULLs
1. Filter the NULL value out
	- ```sql
	   SELECT * FROM 
		   (
			   SELECT month, visitors,
				   visitors - LAG(visitors) OVER (ORDER BY month) AS mom_change
				FROM monthly_arrivals
			) t
		WHERE mom_change IS NOT NULL
	   ```
2. Use a DEFAULT value
	- ```SQL
	  SELECT 
		  month, revenue, 
		  revenue - LAG(revenue, 1, 0) OVER (ORDER BY month) AS change 
		FROM monthly_revenue;
	  ```
3. Use COALESCE
	- ```sql
	WITH changes AS 
		( 
			SELECT 
				month, revenue, 
				revenue - LAG(revenue) OVER (ORDER BY month) AS change 
			FROM monthly_revenue 
		) 
	SELECT 
		month, revenue, 
		COALESCE(change, 0) AS change_display 
	FROM changes;
	  ```