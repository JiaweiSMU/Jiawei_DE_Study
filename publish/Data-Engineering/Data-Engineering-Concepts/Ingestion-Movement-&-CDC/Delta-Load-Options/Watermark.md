A watermark is the ==**stored position**== up to which source changes have already been applied to the target. 
An incremental extract uses it as a bookmark: ==each run reads everything after that position, applies it, then moves the watermark forward.==
- The position comes from the **source's own data or clock**, not from when the pipeline ran.
### Types

| Watermark | Catches | Misses |
|---|---|---|
| `updated_at` (timestamp) | Inserts and updates | Deletes; writers that don't set `updated_at` |
| `created_at` (timestamp) | Inserts only | Updates and deletes |
| Increasing ID (last `order_id` loaded) | Inserts only | Updates and deletes |
### Low and high watermark
- **Low watermark:** where the last successful run stopped 
- **High watermark:** the upper bound of this run, ==fixed **before** extracting==, taken from the source's clock minus a safety lag.
> [!NOTE] Setting a high watermark is to ensure that:
> - **Fixed window:** a rerun covers exactly the same range.
> - Just in case rows are still being written during the extract


4. **Apply** to the target: deduplicate to the latest version per key, then `MERGE` by key. The apply must be idempotent, so re-reading a row does no harm.
5. **Advance** the watermark to the high value, only after step 4 has committed. If the window returned zero rows, still advance to the high value: the window was read, it just held no changes.
### How it works
1. Read the low watermark
```sql
SELECT watermark FROM etl_control.watermarks WHERE source_table = 'orders';
-- 2026-10-17 02:00:00
```
2. Fix the high watermark on the source's clock, minus a safety lag
```sql
SELECT now() - interval '5 minutes';   -- run on the source
-- 2026-10-18 01:55:00
```
3. Extract based on the bounded window
```sql
SELECT * FROM orders
WHERE updated_at >  '2026-10-17 02:00:00'   -- low
  AND updated_at <= '2026-10-18 01:55:00';  -- high
```
4. **Apply** to the target: ==deduplicate to the latest version per key==, then `MERGE` by key. The apply must be idempotent, so re-reading a row does no harm.
5. **Advance** the watermark to the high value, only after step 4 has committed. If the window returned zero rows, still advance to the high value: the window was read, it just held no changes.
Generally it's good to run both **Step 4 & 5** in ==a single transaction==, so that they either commit together or rollback together.



