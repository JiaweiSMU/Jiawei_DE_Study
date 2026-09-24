> [!NOTE] Interview Pattern Cheat Sheet
> - Top N per group: `RANK`/`ROW_NUMBER` + `PARTITION BY`, filter in outer query
> - Running total: `SUM() OVER (ORDER BY …)`
> - Period-over-period: `LAG()` for previous value, calculate difference
> - Moving average: `AVG() OVER (ROWS BETWEEN n PRECEDING AND CURRENT ROW)`
> - Percentile: `NTILE(100)` or `PERCENT_RANK()`
> - Consecutive streaks: `ROW_NUMBER()` + date arithmetic