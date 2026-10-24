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
The engine finds that order 4217 is at position 2 in `file-b`. It **does not touch `file-b`**. Instead it writes two small new files:
- **Delete file**: Mark specific rows (File & Position) as removed from the table, so readers will skip them
- **New data file**: It contains the new version of the changed row
```text
delete-1.parquet        (a delete file: "these rows no longer count")
file    position
file-b  2

new-1.parquet           (a normal data file with the new version of the row)
order_id  region  amount
4217      SG      60.00
```
Then it commits a new version whose metadata lists:
```text
v6:  file-b + delete-1 (applies to file-b) + new-1
```
### The read: this is the “merge”
A query reads `v6`. To get the correct rows, the engine merges the three files as it reads:
1. Read `delete-1`: position 2 of `file-b` is deleted.
2. Read `file-b`, **skipping position 2**: you get 4215, 4216, 4218, 4219.
3. Read `new-1`: add 4217 with amount 60.00.
```text
Result
order_id  region  amount
4215      SG      20.00
4216      MY      35.00
4218      TH      15.00
4219      SG      70.00
4217      SG      60.00   ← new value
```
The ==change is merged into the data **when it’s read**== (IN MEMORY, not the actual file), not when it’s written.
### Why this gets slower over time
Suppose 50 more updates hit rows in `file-b` over the day. Now `file-b` has 50 delete files and 50 small new data files attached. **Every query** that reads `file-b` must read all of them and apply all of them. Writes stayed cheap, but the cost has moved to reads, and it grows with each update.
### Compaction cleans it up
A compaction job eventually:
1. reads `file-b` with all its delete files and new rows applied,
2. writes one clean file, `file-b2`, holding only the current rows,
3. commits a version that lists just `file-b2`.
Reads are back to one file and no merging. (The old files stay until the versions listing them are expired.)