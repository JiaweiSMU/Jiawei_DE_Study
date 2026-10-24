Partitioning **divides** a whole (entire data) into ==non-overlapping pieces according to a rule==, so that an operation can **work on one piece instead of the whole**. 
- Every item belongs to **exactly one** piece: none are left out, none appear twice. 
- So if you know the rule, you know which piece an item is in, and which pieces it cannot be in. 
- That lets a system act on one piece without looking at the others.


### Key questions
> [!NOTE] Terminology
> - Data Assets: Piece of data that the pipeline produces and keeps (`table, set of files`)
##### What is being split
1. Storage: Rows of one table
2. Orchestration: **Work** of a **data asset or pipeline** into slices of its data. 
	1. Same code runs for every slice and orchestrator can track each slice's status and can run, re-run or [[Backfill]] one slice on its own.
3. Execution: How a job **splits the data** it's processing into chunks. Each chunk is handled by one task and many **tasks run in parallel** across the cluster cores. 
##### Key to split on
It can be a `column / attribute` that maps each key value to a row in the data
- Range (`order_date` by month):  
- List / Value (`country ='SG'`): 
- Hash: Allows data to be spread evenly
##### Benefits
- Skip data by reading only rows where a filter can match
- Replace data (rebuild, overwrite, or drop) data by matching without touching rest of the data
- Parallelize the data by giving different pieces to different workers at the same time
### Costs of partitioning
##### Skew
When a piece (e.g. holds more data) than the test, it will still result in slowness. As everything still have to wait for the slowest worker
##### Operations that don't use the key get nothing
- Queries with no filter on partition columns, will still need to read every piece of data
### Example:
A daily job builds yesterday’s orders:
1. **Orchestration:** the orchestrator runs partition key `2026-10-23`.
2. **Execution:** Spark reads that day’s files and splits them into many in-memory partitions, one per task. To compute `SUM(amount) BY region`, it shuffles the rows so that each region’s rows land in the same task.
3. **Storage:** the result is written into the `order_date=2026-10-23` partition of the target table.

> [!NOTE] Note
> ==Directory paths are not the partition== itself. A partition is defined by its **==key values==**, recorded in the catalog or table metadata; the path is only where its files are stored.
> _Example_: `year=2026/month=10/day=23/` is one path with three levels.

Way to ==organize data== into **non-overlapping parts**, using a rule based on a key. Each part can then be ==read, rebuilt or processed independently of others==.
- More **performant & Cost efficient**: Operates on a subset of data instead of all at once.
	- Makes [[Backfill]] simpler
_Example_: 
- **Per day partition**: where each corresponds to the set of event records that arrived on a particular day
```text
page_views/ 
	event_date=2026-10-22/ part-0000.parquet → rows 1, 2 
	event_date=2026-10-23/ part-0000.parquet → rows 3, 4 
	event_date=2026-10-24/ part-0000.parquet → row 5
```
### Multi Dimensional Partitions
When the data is partitioned **across two or more dimensions** (e.g. `cust_id, date`) rather than just on a single attribute (e.g. `date`)
- So one is able to refresh the data for a particular customer for a particular date![[Pasted image 20261024151220.png|382]]

Partitioned data pipelines:
- These are just data pipelines that are organized and processes datasets independently for some manageable subsets of data (partition)

### Where partitions get complicated
As data pipelines usually need to combine different data sources all with their own partitions, shape and cadence of updates
- Different data assets might be partitioned differently, and it doesn't make sense to partition all the data assets in the same way
	- _Example_: Daily partitioned table that depends on an hourly partitioned table
- Even when data assets are partitioned in the same way, there might be dependencies where a data asset relies on earlier / later partitions of another data asset.
	- _Example_: Some calculations might require Day on Day so today's partition actually require yesterday one. Same goes for Week on Week


# Fix this
### **Hot partition**
A hot partition is a ==partition that receives a disproportionate share of a topic’s traffic==. If one key is far more active than the rest, all of its events go to the same partition.
- e.g. click events keyed by `ad_id`: a viral ad’s clicks all land in one partition.
Effects:
- The partition’s **leader broker** handles a much larger share of writes and reads than other brokers.
- The partition is read by **only one consumer** in each group, so that consumer falls behind (lag grows) while others sit mostly idle.
- _Adding partitions or consumers doesn’t help, because the hot key still hashes to one partition_.
### How to handle a hot partition
1. **Remove the key.** Events without a key are spread across all partitions. Use this only when ==no per-key ordering is needed==.
2. **Compound key (key salting).** Append a ==random suffix== from a ==fixed range== to the key, e.g. `ad_id:0` to `ad_id:9`. The hot ad’s events are now spread over up to 10 partitions (two suffixed keys can still hash to the same partition).
    - Ordering holds within each compound key, but not across all events for the ad.
    - Anything needing per-ad results (e.g. total clicks per ad) must combine all 10 sub-streams downstream.