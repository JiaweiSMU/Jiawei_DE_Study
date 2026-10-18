A consumer group is a **set of consumers that share one `group.id`** and together **read a topic as one logical reader**.
- Consumers are added to the same group in order to **share the load**
Within the group, ==each partition is assigned to exactly one consumer== so that the **work is not duplicated**.
Example:
```text
Topic "orders": partitions p0, p1, p2, p3

Group "redshift-loader", 2 consumers:
  consumer A ← p0, p1
  consumer B ← p2, p3
```
### Rules
1. Within a group, the topic is shared by partition. Each partition is assigned to one consumer; ==one consumer may hold several partitions==.
2. **[[Across groups, every group reads every event.]]** (Click in for explanation) Groups are **independent** and **don’t see each other**. Each group **keeps its own committed offset per partition** (the next event it will read), so reading by one group doesn’t affect another.

