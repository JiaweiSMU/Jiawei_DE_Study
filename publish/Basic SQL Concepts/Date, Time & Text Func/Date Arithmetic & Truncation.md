### Summary
| Task                           | MySQL                                                     |
| ------------------------------ | --------------------------------------------------------- |
| Today / now                    | `CURDATE()` / `NOW()`                                     |
| Add or subtract time           | `date + INTERVAL 7 DAY`, `DATE_ADD(...)`, `DATE_SUB(...)` |
| Days between                   | `DATEDIFF(later, earlier)`                                |
| Months, hours, seconds between | `TIMESTAMPDIFF(unit, earlier, later)`                     |
| Round to month                 | `DATE_FORMAT(d, '%Y-%m-01')`                              |
| Monthly totals                 | `GROUP BY DATE_FORMAT(d, '%Y-%m')`                        |
| Extract parts                  | `YEAR(d)`, `MONTH(d)`, `DAY(d)`                           |
| **Never**                      | `date1 - date2`                                           |
#### To add / Subtract time
When asked what's the date 30 days from now or 1 month ago.
- Use `INTERVAL` to help calculate
```sql
-- Addition
DATE_ADD(<date_col>, INTERVAL 7 DAY)
DATE_ADD(<date_col>, INTERVAL 1 MONTH)

-- Minus
DATE_SUB(<date_col>, INTERVAL 7 DAY)

-- Alternative
date_col + INTERVAL 7 DAY o
date_col - INTERVAL 30 DAY
```
### For finding gap between 2 dates
`DATEDIFF(<later date>, <earlier date>)`: Return num of days
`TIMESTAMPDIFF(unit, <earlier date> <later date>)`: Return months, years, hours etc.
#### DATEDIFF()
> [!NOTE] 
> `DATEDIFF(later, earlier)`
> - To use LATER date then EARLIER date, else will get a negative value
> - Only works in **DAYS**. 
> - For Months / Years etc. to use `TIMESTAMPDIFF

| Query                                  | Result | Why                                |     |
| -------------------------------------- | ------ | ---------------------------------- | --- |
| `DATEDIFF('2024-03-15', '2024-03-10')` | `5`    | 15th minus 10th                    |     |
| `DATEDIFF('2024-04-01', '2024-03-31')` | `1`    | handles month boundaries correctly |     |
| `DATEDIFF('2024-03-01', '2024-02-01')` | `29`   | knows 2024 is a leap year          |     |
| `DATEDIFF('2024-03-10', '2024-03-15')` | `-5`   | dates swapped, so negative         |     |
#### TIMESTAMPDIFF()
```sql
-- Session length 
TIMESTAMPDIFF(SECOND, session_start, session_end) AS session_sec 

-- Support ticket response time 
TIMESTAMPDIFF(MINUTE, created_at, first_reply_at) AS response_min 

-- Tickets that breached a 4-hour SLA 
WHERE TIMESTAMPDIFF(HOUR, created_at, resolved_at) > 4
```
### Rounding dates down to a period
In SQL the Syntax is:
```sql
-- Search online to find the format
SELECT 
	DATE_FORMAT(<date>, <way to format>) 
```

Rounding down converts a date to the start of its period: the first day of the month, week or quarter. Every date in the same period ends up with the same value, so the dates can be grouped together with `GROUP BY`.

Unlike `MONTH(d)`, the rounded date keeps the year, so the same month in different years stays separate.

**Scenario:** To get monthly totals, grouping by `order_date` doesn't work, because every day becomes its own group. Rounding each date to the first of its month fixes this:

|order_date|amount|
|---|---|
|2024-03-02|50|
|2024-03-15|30|
|2024-03-28|20|
|2024-04-05|40|
