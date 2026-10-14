A GROUP BY query built by dragging fields

![[Pasted image 20261014213844.png]]
- **Rows**: like GROUP BY. One row per unique value. Two fields in Rows = nested grouping (e.g. Region, then Channel inside it).
- **Columns**: a second GROUP BY spread across the top, turning the list into a grid (crosstab).
- **Values**: the aggregate. Right-click a number in the pivot:
    - _Summarize Values By_ changes the calculation (Sum, Count, Average, Max)
    - _Show Values As_ changes the display (% of Grand Total, Running Total, Rank)
- **Filters**: like WHERE. Limits which source rows go in before grouping. Slicers do the same job with buttons.
- The pivot reads a saved copy of the data, so after changing the source, **Refresh** (Alt+F5).
- Double-click any number to see the rows behind it.

### Slicers and Calculated Fields
Both are on the **PivotTable Analyze** tab. That tab only appears when you click a cell inside the pivot.

**Slicer = a filter shown as clickable buttons**
- _How:_ PivotTable Analyze > **Insert Slicer** > tick a field (e.g. Channel) > OK

**Calculated Field = a new column the pivot calculates from other columns**
- _How:_ PivotTable Analyze > **Fields, Items & Sets > Calculated Field…** > Name: `Margin`, Formula: `=Profit/Revenue` > Add > OK
- It works on totals: `SUM(Profit) / SUM(Revenue)` for each row of the pivot

### Running Total 
- **Day by day:** ungroup the dates so each date is its own row
- **Month by month:** group by **Years + Months** (not Quarters). It restarts each January, so it’s a **year-to-date** figure
- **One total that never restarts:** add a helper column `=TEXT(OrderDate,"yyyy-mm")` and use that in Rows
- **% Running Total In** = the running total as a % of the overall total (e.g. “top 5 customers = X% of revenue”)

**% Difference From** (e.g. this year vs last year)  
Show Values As > % Difference From has 2 settings. 
Read them as: Compare each **Base field with Base item**.
1. **Base field**: the field whose values are compared with each other (Years, Months, Region…). It must already be in Rows or Columns.
2. **Base item**: which value of that field you compare against.
    - _(previous)_ = the one before, e.g. Base field Years → this year vs last year
    - _(next)_ = the one after
    - or a specific value, e.g. _Singapore_ → every region vs Singapore