Data Schema has to be pre-defined ([[Schema On Write]]) and transformation completed before it reaches the final destination. 
Whereas [[ELT (Extract, Load, Transform)]], allows you to load the data first before transforming it.
### Extract
The extract job connects to someone and runs a `SELECT`. 
- The query reads only ==row versions committed when its snapshot was taken==. Changes committed later are ignored. 
#### Full extract
Read the whole table on every run.
- **Why use it:** simple and always correct. Deletes are handled, because the whole table is replaced.
- As size of table grows, it might take too long to perform a full extract of the data
#### Incremental extract with a watermark
Read only ==new / modified records== from the source system ==since the last successful run== instead of copying the entire dataset again.
- **Watermark:** This can be some timestamp, ID etc. to indicate what is the **latest point of data that was previously processed**
- **Common watermark columns:**
  - a timestamp (`modified_date`): catches inserts and updates
  - a sequential ID (`id > x`): catches **inserts only**, because an updated row keeps its ID
**Issues being:**
1. **Deletes are invisible.** A deleted row matches no query, so it stays in the warehouse forever.
2. If the source doesn't have a reliable way to track datetime (`modified_date`) you might be left with stale data
### Transform
This is where the cleaning happens, joining of tables and general transformation to get it ready for analytics use
### Load
**Goal of every load pattern: idempotency.** Pipelines fail and get retried, so ==running the same load twice must give the same result as running it once==.

**Load Patterns:**

| Pattern               | How it works                                                                                                                            | Failure mode                                                                                                                                               |
| --------------------- | --------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Append**            | `COPY` new rows into the target                                                                                                         | A **rerun after a partial failure duplicates rows**                                                                                                        |
| **Truncate & reload** | Empty the table, then `COPY` everything                                                                                                 | In Redshift, `TRUNCATE` commits immediately, so readers see an empty table until the load finishes. <br><br>Fix: load into a new table and swap the names. |
| **Upsert (merge)**    | `COPY` into a staging table → `DELETE` target rows whose keys appear in staging → `INSERT` from staging, all in ==**one transaction**== |                                                                                                                                                            |
