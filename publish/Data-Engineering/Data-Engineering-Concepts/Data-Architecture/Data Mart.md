A data mart is a subset of the [[Data Warehouse]] scoped to ==one business domain== (e.g. finance, sales), with tables shaped for that domain's reports.
### Why it exists
- **Simpler queries**: an analyst sees only the tables for their domain (e.g. 5 tables instead of 50).
- **Access control**: permissions are granted per mart, so each department reads only its own area.
- **Speed**: mart tables are often ==pre-joined or pre-aggregated== for that **domain**'s common reports.



