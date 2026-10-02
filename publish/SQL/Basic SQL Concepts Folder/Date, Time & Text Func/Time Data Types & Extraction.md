> [!NOTE] 
> `TIMESTAMP` column can't be compared against a `DATE` column without casting. It won't match as `TIMESTAMP` column carries a time component.
> 
> To cast it:
> ```sql
> -- AS DATE here refers to (DATE TYPE)
> WHERE CAST(date AS DATE) 
> ```
#### Date / Time Data Types
- `DATE`: stores the date only (year, month, day), e.g. `2024-03-15`
- `TIME`: stores the time only (hours, minutes, seconds), e.g. `14:30:00`
- `TIMESTAMP` (called `DATETIME` in MySQL / SQL Server): stores both date and time, e.g. `2024-03-15 14:30:00`
#### Current Date & Time
 - Today's date: `CURRENT_DATE`
 - Current date and time: `NOW()/CURRENT_TIMESTAMP`
 - Current time: `CURRENT_TIME` returns the current time of day (hours, minutes, seconds) with no date component.
#### ==Extracting parts of a date==
1. Year: `EXTRACT(YEAR FROM <date>)`
2. Month: `EXTRACT(MONTH FROM <date>)`
3. Day: `EXTRACT(DAY FROM <date>)`
```sql
-- To find employees who joined in Jan
Select
	emp_name,
	join_date
FROM employees
WHERE EXTRACT(MONTH FROM join_date) = 1
-- Alternative is
-- WHERE MONTH(join_date) = 1
```