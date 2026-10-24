Way to ==organize data== into **non-overlapping parts**, using a rule based on a key. Each part can then be ==read, rebuilt or processed independently of others==.
- More **performant & Cost efficient**: Operates on a subset of data instead of all at once.
	- Makes [[Backfill]] simpler



To organize data and the computations that produce that data ??
- Makes data pipeline **more performant and cost efficient** 
	- Operate on **subset of data** instead of all of it at once

Table, can be built up of a few partitions.
Each partition can be a separate file or a slice of data in a database (e.g. yyyy-mmm)
_Example_: 
- Per day, where each partition corresponds to the set of event records that arrived on a particular day
- Each country (e.g. Singapore, Malaysia) they are all a separate partition. Within it of course one can still break it down into per day etc. (Also known as multi dimensional partitions)

### Multi Dimensional Partitions
When a data asset is partitioned across two or more independent dimensions rather than just a single attribute (e.g. `date`)

One example would be to partition across `customer_id` and `date`
- So one is able to fresh the data for a particular customer for a particular date![[Pasted image 20261024151220.png|382]]

Partitioned data pipelines:
- These are just data pipelines that are organized and processes datasets independently for some manageable subsets of data (partition)

### Where partitions get complicated
As data pipelines usually need to combine different data sources all with their own partitions, shape and cadence of updates
- Different data assets might be partitioned differently, and it doesn't make sense to partition all the data assets in the same way
	- _Example_: Daily partitioned table that depends on an hourly partitioned table
- Even when data assets are partitioned in the same way, there might be dependencies where a data asset relies on earlier / later partitions of another data asset.
	- _Example_: Some calculations might require Day on Day so today's partition actually require yesterday one. Same goes for Week on Week



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