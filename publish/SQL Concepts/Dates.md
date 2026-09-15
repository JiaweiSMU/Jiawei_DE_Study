### Difference between Date & Timestamp
`DATE` refers to calendar days only, in `YYYY-MM-DD` format.  
![[Pasted image 20260915224147.png]]

`TIMESTAMP` refers to a precise moment - it stores both date and time.  
![[Pasted image 20260915224147.png]]
### Comparison
If you compare a `timestamp` column against a `DATE` value without casting, it usually won't match, since the timestamp still carries a time component the date doesn't have. Cast the timestamp down to `DATE` first to strip the time and compare day to day.  ![[Pasted image 20260915224446.png]]
### Current Date & Time
- `CURRENT_DATE` returns just today's date.
- `CURRENT_TIME` returns the current time of day (hours, minutes, seconds) with no date component.
- `CURRENT_TIMESTAMP` returns both date and time together.
```sql
-- Finding users who logged in within the last 24 hours
SELECT
    user_id,
    last_login
FROM users
WHERE last_login >= CURRENT_TIMESTAMP - INTERVAL '24' HOUR
```
Use `INTERVAL` when comparing against something like "x hours ago" or "x days from now."
### ==DATE_ADD()==
Adds or subtracts a time span to a date or timestamp and returns a new date/timestamp - does the same job as `date_col + INTERVAL '7' DAYS`.
```
Syntax: DATE_ADD(unit, amount, date)
- Unit: Day / Month / Year / Hour etc.
- To deduct, set the amount as '-7'
```
### ==DATE_DIFF()==
Calculates the difference between two dates/timestamps in a specified unit, returning an integer.
```
Syntax: DATE_DIFF(unit, start_date, end_date)
- Unit: Day / Month / Year / Hour etc.
```
> [!Note] 
> `DATE_DIFF(day, '2024-01-01', '2024-01-10')` returns `9`. Flip the order and you get `-9` instead.
> 
To note: `DATE_DIFF` counts complete calendar units crossed, not elapsed time. An order placed at 11PM and shipped at 1AM the next day only has 2 hours between them, but `DATE_DIFF(day, ...)` still returns `1`, since the two timestamps fall on different calendar days.

