> [!NOTE] Note
> Backfill is a **computationally heavy process**, require lots of memory
> 
> Good link to read:
> - https://dagster.io/blog/backfills-in-ml

**Re-running a pipeline for past periods**, either to ==fill periods== that were never processed or to ==fix periods== that were processed wrongly.
It is needed when re-running the pipeline on today’s source (e.g. the latest extract) would not reproduce the table’s history. This happens when:
- **The table is built one period at a time** (an incremental pipeline), so its history comes from many past runs, not one.
- **The source no longer holds the old data**, because rows were overwritten, archived or deleted.
### When to backfill
1. **Greenfield**: ==New table== that is added to the pipeline requires **historical data**. This is a fill as **nothing is replaced.**
	- Example: `customer_lifetime_value` is created today, but analysts need it from 2020, so the pipeline runs for every month since 2020.
2. **Brownfield**: Any changes that makes existing history wrong / incomplete. This is a restatement; **existing data is replaced**.
	- A calculation that used columns `a`, `b` and `c` now also needs `d`.
	- A new column must be filled for past periods.
	- The source corrects past records.
3. **Recovering from failure**: Periods are missed / Half written
	- Example: the job failed from 3 to 7 October. Re-run those days, replacing anything the failed runs partly wrote.
### What backfill needs to work
1. **The ==raw input== for those periods still exists.** Keeping only the transformed output is not enough.
	- A backfill produces a **different output from the current run**.
		- Example: the latest output holds monthly totals, and a bug miscounted refunds. The totals cannot be corrected without the individual orders they were summed from.
2. **Re-running a period ==replaces it (idempotency)==.** Running a period twice leaves the table as if it ran once.
	- For 2023Q4: delete 2023Q4 and insert it again in one transaction, or overwrite the 2023Q4 partition.
	- Appending would count 2023Q4 twice.
3. **The run depends only on the period it is given.** No `current_date` or “last 30 days” in the logic.
### [[Partition]]ing
Partitioning the table by period, e.g. by hour, makes the **partition the unit of a backfill** and only **overwriting the affected partitions**.
- **What to backfill is easy to find.** Each partition can be checked on its own: is it missing, or was it built with outdated logic?
- **Replacing is one operation.** Overwriting a partition replaces exactly that period, idempotently and, in table formats like Iceberg, atomically.

Processing **one partition at a time**, rather than all the data in one run, also makes the backfill:
- **Fault-tolerant:** a **failure** costs only the **partition being processed**. Re-run that one; finished partitions are untouched. This holds even for a single-threaded script.
- **Parallelizable:** independent partitions can **run at the same time**, limited by what the source and the warehouse can handle.
Trade-off: partitions that are too small create many tiny files and many commits.
### Before backfilling
- Define the **exact range of periods** to backfill.
- **Corrections applied directly to the table ==are lost** when the backfill replaces== those rows.
	- Store corrections in their own table and apply them inside the transformation, so every recompute includes them.
### During and after backfilling
- ==**Swap atomically**==, so readers never see a mix of old and new data.
- **Refresh everything that stores results built from the backfilled periods.**
    - Example: `orders` is backfilled for October. Re-run `monthly_revenue` for October, plus later periods that use October (year-on-year, rolling averages).
    - Then refresh anything built from `monthly_revenue`, and so on downstream.
### How backfilling can go wrong
- Fail to backfill parts of the data that needs to be backfilled, the data might look up to date but it's not
- **Backfill unneeded periods:** their numbers change unexpectedly under new logic, and direct corrections are wiped.
- **Derived tables not refreshed:** the backfilled table and the tables built from it disagree.
- **Partial failure:** some periods use the new logic and some the old, with no way to tell which.