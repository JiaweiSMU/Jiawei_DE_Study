Partitioning **divides** a whole (entire data) into ==non-overlapping pieces according to a rule==, so that an operation can **work on one piece instead of the whole**. 
- Every item belongs to **exactly one** piece: none are left out, none appear twice. 
- So if you know the rule, you know which piece an item is in, and which pieces it cannot be in. 
- That lets a system act on one piece without looking at the others.
> [!NOTE] Terminology
> - Data Assets: Piece of data that the pipeline produces and keeps (`table, set of files`)
> - Partition / Piece: One of the subset created by the split
> - Partition Key: Column / Attribute the rule is applied to (To choose the split)
### How partitioning works
##### 1. What is being split
1. [[Storage Partitioning]]: Rows of one table
	- e.g. `orders` split by month of `order_date`
2. [[Orchestration Partitioning]]: The job of building the data asset or pipeline, is split into many small run
	- **Declare how the data asset is partitioned** (e.g. daily from 2026-01-01). From this, the **orchestrator** knows every ==partition key== and passes one key to each run. The job takes the partition key as input.
		- With the orchestrator being able to take partition key as input, one can re-run one key or [[Backfill]] a range of keys.
	- It records the ==status of each partition key ==(built, failed, not built; when; by which run). 
3. [[Execution Partitioning]]: How a job splits the data it's processing into chunks. Each chunk is handled by one task, and many tasks run in parallel across the cluster cores
##### 2. Rule: _Key + Function_
==Key== (`column / attribute`) and a ==function== that maps each key value to a partition

| Function     | Example                    | Strength                                                         | Weakness                                                                                         |
| ------------ | -------------------------- | ---------------------------------------------------------------- | ------------------------------------------------------------------------------------------------ |
| Range        | `order_date` by month      | A range filter ("since January") touches few partitions          | New rows all carry the current date, so all writes land in the newest partition.                 |
| List / value | `country = 'SG'`           | Partitions match how people think about the data                 | Partition sizes follow the real distribution: one large country gives one large partition        |
| Hash         | `hash(customer_id) mod 16` | Spreads data evenly; an exact-match lookup goes to one partition | A range filter must touch every partition, because neighbouring values are scattered across them |
##### 3. How the system knows which partition holds what
The system needs a **map from each partition to its data or status**, and it reads this map **before** doing any work. 
- Map: Holds one entry per partition (`key value` of the partition)
	- Based on the request (`query filter / keys to run`), the system refers to the map to decide what partitions to touch

Per Layer:

| Layer         | What the record is                                                                                                                                 | Who reads it, and when                                                         |
| ------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------ |
| Storage       | A catalog / table metadata that maps each partition to its data (e.g. `.../order_date=2026-01-15/`)                                                | The query planner, before reading any data, to decide which partitions to skip |
| Orchestration | Stores the current status of each partition key: built, failed, not built; when; by which run                                                      | The orchestrator, to decide which keys to run<br>You, to see what is missing   |
| Execution     | Nothing is stored. This contains the engine execution plan, the rule itself (e.g. `hash(region) mod 200`), and each task computes where a row goes | Every task during the run                                                      |
##### 4. What performance are improved
The benefits are a result of step 3: The system **chooses partitions from the map, ==without reading the data.==**
- Skip data by **reading only the partition whose key value can match the filter**
	- Skipping happens at the partition level. Rows inside the partition that are read are still filtered one by one
- **Replace**: rebuild, overwrite or drop one partition, identified by its key value, without touching the rest
	- e.g. drop the storage partitions older than 2 years, or re-run one day's orchestration key
- **Parallelize**: give different partition of the **work** to different workers at the same time
	- e.g. a Spark job splits its data into 200 execution partitions, each processed by one task
##### 5.Costs of partitioning
###### Skew
When one partition holds much more than the others.
	- Situation: `orders` partitioned by day, where the 11 Nov sale day holds 20× the orders of a normal day
- **Storage**: partition sizes become uneven. One date has more files than the others and a query on that day needs to read more data as compared to other days.
- **Orchestration**: run durations become uneven. The run takes much longer than other days' runs, so it can exceed a timeout or need more compute than the job was sized for
- **Execution**: a job finishes only when its last task finishes. If one task gets the oversized partition, the other workers finish early and sit idle while it runs
###### Operations
Work that doesn't filter or group on the key can't use the record, so it touches every partition
- **Storage**: Query with no filter on the partition column **reads every partition**.
- **Execution**: a join or group-by on a different column forces a [[Shuffle]]
- **Orchestration**: a change to the logic means backfilling every partition key


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