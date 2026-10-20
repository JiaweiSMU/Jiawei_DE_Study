A consumer group is a **set of consumers that share one `group.id`** and together **read a topic as one logical reader**.
- Consumers are added to the same group in order to **share the load**

Within the group, ==each partition is assigned to exactly one consumer (one consumer may hold several partitions== (read from those partitions) so that the **work is not duplicated**.
_Example_:
```text
Topic "orders": partitions p0, p1, p2, p3

Group "redshift-loader", 2 consumers:
  consumer A ← p0, p1
  consumer B ← p2, p3
```
### Rules
1. Within a group, the topic is shared by partition. Each partition is assigned to one consumer; one consumer may hold several partitions.
2. **[[Across groups, every group reads every event.]]** (Click in for explanation) Groups are **independent** and **don’t see each other**. Each group **keeps its own committed offset per partition** (the next event it will read), so reading by one group doesn’t affect another.
### Committed Offsets
Within a consumer group, the committed offset for each partition is the **official record of how far the group has processed that partition**. 
Consumers **send it to the group's coordinator**, the broker in charge of that group's offsets, which stores it in `__consumer_offsets`. 
- It is the **next** offset the group will read in that partition.
![[Pasted image 20261018231857.png]]
- If a consumer crashes, a **rebalance** gives its partitions to other consumers in the group. Each new owner ==fetches the committed offset from the coordinator and continues from there==.
- Events **processed after the last commit are processed again** by the **new owner**. 
##### consumer_offsets
It's an internal Kafka topic that stores **consumer groups'** committed offsets keyed by `group + topic + partition`. 
Each commit writes a record to it, and **compaction** ==keeps only the latest offset for each key==.
```text
key:   (group = redshift-loader, topic = orders, partition = 2)
value: offset 81,200
```
- One partition holds the offsets of **many consumer groups**.
- For a consumer group, ==all the topic and partitions go to the same consumer_offsets partition==
##### Group Coordinator
The broker that **leads a `__consumer_offsets` partition** is the **coordinator for every group stored in it**.
- Each group has **one** coordinator; a broker can coordinate **many** groups.
- It handles the group's **membership** (joins, heartbeats, rebalances) and its **offset commits**.
_example_:
```text
group "redshift-loader" → __consumer_offsets-17 → leader broker 2 → coordinator: broker 2
group "fraud-check"     → __consumer_offsets-4  → leader broker 1 → coordinator: broker 1
group "email-service"   → __consumer_offsets-33 → leader broker 2 → coordinator: broker 2
```
### Offset Commit Timing
Whether a consumer **commits its offset** **before** or **after** processing, decides what happens to **in-progress events if it crashes**: they are either **processed again** or **skipped**.
- **Commit after processing:** a crash before the commit means those ==events are processed again==. Duplicates, no loss, so the sink must handle repeats.
- **Commit before processing:** a crash before processing finishes means those events are skipped. No duplicates, but loss.
_example_:
```text
p0:  ... [100] [101] [102] [103] [104] ...
	            ▲             ▲
      committed offset: 101     consumer A had processed up to 103, then crashed

Consumer B takes over p0 and starts at 101.
Events 101, 102 and 103 are processed a second time.
```
### Scaling
The **maximum number of consumers is usually equals to the maximum number of partitions**. If you've more consumers than partitions, there will be **idle consumers**.
### Rebalance
When a consumer joins, leaves or stops responding, the group's partitions are **reassigned across its remaining members**.
```text
Before:  A ← p0, p1     B ← p2, p3

B crashes

After:   A ← p0, p1, p2, p3
```
