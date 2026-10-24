Hash-based incremental loading pattern when there's no other way to detect changes (no reliable `updated_at`, and no access to the WAL or to triggers.)
### Requirements
- A ==**stable key** on both sides== (primary key, business key, or composite key) that identifies the same entity on every run. 
	- The key tells you *which* rows to compare
	- The hash tells you *whether* they differ.
- Without a stable key, you can only hash whole rows: an update then looks like a delete plus an insert, and exact duplicate rows must be compared by count.
### How it works
1. **Full scan** of the source table on every run.
2. ==**Hash the non-key columns** of each row==. Keep the key unhashed, because it is the join condition.
	- ==Exclude pipeline-added columns== (for example a load timestamp), or every row looks changed.
3. **Join on the key** against the target, which stores its own hash per row.
	- One [[Full Outer Join]] classifies everything in one pass.
	- Or, **anti joins for inserts and deletes**, plus an inner join with `hash <> hash` for updates.
4. **Classify the changes**
	- Insert: key in source, not in target
	- Update: key in both, hashes differ
	- Delete: key in target, not in source
### Pros & Cons

| Type | Point                                    | Why                                                                                                                                                              |
| ---- | ---------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Pro  | Simple to write                          | Plain SQL: a hash expression and one join                                                                                                                        |
| Pro  | Catches inserts, updates **and deletes** | A key missing from the source is a delete; a key missing from the target is an insert                                                                            |
| Pro  | Needs nothing from the source            | Read access only; works on CSV extracts and API dumps too                                                                                                        |
| Con  | Cost grows with table size               | Full source scan every run                                                                                                                                       |
| Con  | ==Comparison is expensive==              | A full join of source against target; the ==hash table may spill to disk==, and rows are redistributed across nodes unless both sides share `DISTKEY` on the key |
| Con  | Only the latest state                    | Intermediate changes between runs are not seen                                                                                                                   |
| Con  | False updates                            | The two sides format values differently (`60` vs `60.00`, timestamp precision, `t` vs `true`); compute both hashes in the same engine                            |
### Reducing the cost
- Store `row_hash` in the target, so the comparison reads two columns (key and hash) instead of all of them.
- Compare one aggregate hash per partition first, and do the row-level comparison only for partitions whose hash differs.