Identifies the ==row-level changes (inserts, updates and deletes)== committed in a **source database**, so a downstream system **processes only what changed** instead of copying the whole table. 
- CDC is the capture step; the changes still need a transport (e.g. Kafka, S3 files) and an apply step (e.g. `MERGE`) to reach the destination.

**Without CDC: full extract (`SELECT *`)**
- Copies the whole table on every run, which handles deletes automatically.
- Cost grows with table 
**CDC methods**
1. Watermark, refer to [[ETL (Extract, Transform, Load)]] for more info:
    - ==Misses deletes== (a deleted row matches no query).
    - Loses intermediate states (three updates between runs appear as one).
    - Can miss rows that commit after the run starts but carry an earlier timestamp.
    - Depends on every writer setting `updated_at`.
2. [[Trigger-based]]: a trigger writes each change to an audit table in the same transaction. Captures every change, but doubles the write work of every transaction.
3. [[Log-based (Write Ahead Logs)]]: reads the database’s transaction ==write ahead log==, which records every committed change in commit order. Captures deletes and every intermediate state with no extra work per transaction.

