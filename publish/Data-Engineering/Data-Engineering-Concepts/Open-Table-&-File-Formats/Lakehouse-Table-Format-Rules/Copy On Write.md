**Starting point**
We want to update the `amount` where `order_id` = 4217
```text
file-b.parquet
pos  order_id  region  amount
0    4215      SG      20.00
1    4216      MY      35.00
2    4217      SG      50.00   ← we'll change this one
3    4218      TH      15.00
4    4219      SG      70.00
```
**The current version is:**
```text
v5:  file-a + file-b + file-c
```
### The write
1. Find which files contain matching rows, in order to open it.
	- The engine checks the min/max stats in the metadata. If `file-a` holds `order_id` 1000–2000 and `file-c` holds 5000–6000, they’re skipped without being opened. Only `file-b` (4215–4219) needs to be opened.
2. Read the whole of `file-b` into memory
3. Apply the change in memory: row 4217 gets amount 60.00.
4. Write a complete new file, `file-b2`, containing **all rows**: the unchanged ones copied as they were, plus the changed one.
```text
file-b2.parquet
pos  order_id  region  amount
0    4215      SG      20.00   ← copied unchanged
1    4216      MY      35.00   ← copied unchanged
2    4217      SG      60.00   ← changed
3    4218      TH      15.00   ← copied unchanged
4    4219      SG      70.00   ← copied unchanged
```
5. Commit a new version (==metadata==), that swaps the old file for the new one:
```text
v6:  file-a + file-b2 + file-c       (file-b is no longer listed)
```
### The read
A query on `v6` reads `file-a`, `file-b2` and `file-c`. There are no delete files to check and nothing to merge. Every file is complete and current, so reads are as fast as if the table had never been updated.
### What happens to `file-b`
It isn’t deleted. It’s still in storage and still listed by `v5`, so:
- a query that started on `v5` before the commit keeps reading it safely
- time travel to `v5` still works
Once `v5` is expired, nothing lists `file-b` and it can be deleted.
### The cost: write amplification
To change **one value**, the engine rewrote the **whole file**. 
- So the ==cost depends on **how many files the changed rows are spread across**==. 
- Rewriting files also raises the chance of commit conflicts: if another writer changed `file-b` first, this writer’s commit is refused and the whole rewrite is wasted.