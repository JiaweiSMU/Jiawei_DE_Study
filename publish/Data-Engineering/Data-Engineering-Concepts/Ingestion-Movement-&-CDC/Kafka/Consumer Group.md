A consumer group is a **set of consumers that share one `group.id`** and together **read a topic as one logical reader**.
- Consumers are added to the same group in order to **share the load**
Within the group, ==each partition is assigned to exactly one consumer (one consumer may hold several partitions== so that the **work is not duplicated**.
Example:
```text
Topic "orders": partitions p0, p1, p2, p3

Group "redshift-loader", 2 consumers:
  consumer A ← p0, p1
  consumer B ← p2, p3
```
### Rules
1. Within a group, the topic is shared by partition. Each partition is assigned to one consumer; one consumer may hold several partitions.
2. **[[Across groups, every group reads every event.]]** (Click in for explanation) Groups are **independent** and **don’t see each other**. Each group **keeps its own committed offset per partition** (the next event it will read), so reading by one group doesn’t affect another.
### Offsets
**Committed offsets** (Next offset the group will read in that partition) in a consumer group are stored ==per group, per partition==, on the brokers.![[Pasted image 20261018231857.png]]
- If a consumer crashes, a **rebalance** **reassigns its partitions to other consumers in the group**. The consumer ==taking over a partition fetches the group’s committed offset from the group== coordinator and continues from there.
```text
p0:  ... [100] [101] [102] [103] [104] ...
            ▲                    ▲
      last commit: 101     consumer A had processed up to 103, then crashed

Consumer B takes over p0 and starts at 101.
Events 101, 102 and 103 are processed a second time.
```
- Events can ==get processed again==, if the committed offset differs.
### Scaling
The **maximum number of consumers is usually equals to the maximum number of partitions**. If you've more consumers than partitions, there will be **idle consumers**.
### Rebalance
When a consumer **joins / leaves the group**, the ==partitions are reassigned among the remaining members==.
```text
Before:  A ← p0, p1     B ← p2, p3

B crashes

After:   A ← p0, p1, p2, p3
```
