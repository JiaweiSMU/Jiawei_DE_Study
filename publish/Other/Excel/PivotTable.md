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