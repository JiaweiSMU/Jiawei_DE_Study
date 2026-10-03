It's only available for [[Data Lake Table Format]].
Time travel allows one to access historical version of the data because `data files` are never modified and `metadata` records the version itself
### Key Features
- **Data Auditing**: To track and review historical changes or just to view historical records
- **Reproducibility**: To ==access specific data version== for reproducing models & experiments
- **Undoing a bad write**: Use to ==`Rollback`== bad writes / accidental deletes
### How to use
```sql
-- Delta (Spark) 
SELECT * FROM orders VERSION AS OF 1; 
SELECT * FROM orders TIMESTAMP AS OF '2026-10-01 09:00:00'; 

-- Iceberg (Athena) 
SELECT * FROM orders FOR VERSION AS OF 4821937465; 
SELECT * FROM orders FOR TIMESTAMP AS OF TIMESTAMP '2026-10-01 09:00:00 UTC';
```
- Accessed via `TIMESTAMP / VERSION`
### Downside
Total storage grows, as files that are ==replaced are kept instead of deleted==.
Have to decide between how far back to keep and how much storage to pay for.