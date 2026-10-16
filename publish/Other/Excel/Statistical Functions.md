## Statistical Functions

### 1. Describing data
| Excel       | SQL equivalent                                 | What it gives you                     |
| ----------- | ---------------------------------------------- | ------------------------------------- |
| `COUNT`     | `COUNT(col)`                                   | how many numbers                      |
| `AVERAGE`   | `AVG(col)`                                     | the mean                              |
| `MEDIAN`    | `PERCENTILE_CONT(0.5)`                         | the middle value                      |
| `MODE.SNGL` | `GROUP BY col ORDER BY COUNT(*) DESC LIMIT 1`  | the most frequent value               |
| `MIN`/`MAX` | `MIN`/`MAX`                                    | smallest / largest                    |
| `STDEV.S`   | `STDDEV_SAMP(col)`                             | spread, sample (default choice)       |
| `STDEV.P`   | `STDDEV_POP(col)`                              | spread, when you have the whole group |
- Mean vs median: a few huge values pull the mean up, so the median is the better "typical" value for skewed data.
- COUNT counts numbers only; COUNTA counts any non-empty cell (including the header).

**Standard deviation, step by step**
1. Work out the mean
2. For each value: (value − mean)², then add them all up
3. Divide → this is the variance
   - STDEV.P: ÷ n
   - STDEV.S: ÷ (n − 1)
4. Take the square root → standard deviation

Use STDEV.S by default (data is a sample). Use STDEV.P only when you have every member of the group.

### 2. With conditions: the -IFS functions
| Excel               | SQL                    | Arguments                              |
| ------------------- | ---------------------- | -------------------------------------- |
| `SUMIFS`            | `SUM(col) WHERE …`     | what to sum, then condition pairs      |
| `AVERAGEIFS`        | `AVG(col) WHERE …`     | what to average, then condition pairs  |
| `MAXIFS` / `MINIFS` | `MAX`/`MIN WHERE …`    | what to max/min, then condition pairs  |
| `COUNTIFS`          | `COUNT(*) WHERE …`     | condition pairs only                   |

**Rule:** with S, the first argument is *what to calculate*, then pairs of *where to look, what to match*.
COUNTIFS is the exception: it has nothing to calculate, so it starts with the pairs.

```
=SUMIFS(Sales!N:N,  Sales!F:F, "Singapore",  Sales!G:G, "Wholesale")
        └ what to sum  └ where to look, what to match (repeat per condition)
```

Without S (`SUMIF`, `AVERAGEIF`, `COUNTIF`): one condition only, and what to calculate goes **last**:
`=AVERAGEIF(Sales!F:F, "Malaysia", Sales!N:N)`

### 3. Writing conditions
| You want            | Write it as                                                 |
| ------------------- | ----------------------------------------------------------- |
| equals text         | `"Malaysia"`                                                |
| bigger / smaller    | `">5000"`, `"<=5000"`                                       |
| not equal           | `"<>Retail"`                                                |
| compare with a cell | `">"&A2`                                                    |
| on/after a date     | `">="&DATE(2025,1,1)`                                       |
| before a date       | `"<"&DATE(2026,1,1)`                                        |
| between two dates   | use two pairs on the same column: on/after AND before       |
| starts with / contains | `"Sing*"`, `"*Home*"`                                    |

- Operators always go inside quotes; join them to a cell or date with `&`
- All conditions are combined with AND. For OR, add two formulas together

### 4. Statistics with no -IFS version (median, stdev, percentile)
Use an **array formula**: build the filtered list yourself, then pass it in.
```
=MEDIAN( IF(Sales!G2:G599="Online", Sales!N2:N599) )
         └─ inner: row by row, makes a list {907, FALSE, 356, FALSE, …}
└─ outer: works on the list, ignores the FALSEs
```
- Don't add `,0` to the IF: MEDIAN ignores FALSE but would count the zeros
- Use exact ranges (G2:G599), not whole columns
- Excel 2016/2019: press Ctrl+Shift+Enter. Excel 365: Enter is fine, or use `=MEDIAN(FILTER(Sales!N2:N599, Sales!G2:G599="Online"))`