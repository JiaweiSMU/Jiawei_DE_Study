[Kafka TLDR](https://www.youtube.com/watch?v=DU8o-OTeoCc)
Kafka is a event streaming platform that **stores events in ==append-only logs==, one log per partition**.
- Producers write ==events to a **Topic**, each event is appended to the one **partition** of that topic== chosen by the event `Key`. All events with the same key land in the same partition and stay in order.
- Consumers read at their own pace, each [[Consumer Group]] tracking its own offset per partition. A consumer can **read several partitions across several topics,** but **within a group** ==each partition has only one consumer==. Reading doesn’t delete events; retention (time or size) does.
- Designed for **many ==small events==**: a record batch is limited to about 1 MB by default.
### Terminology
```text
Data                                    Machines
topic                                   cluster
 └─ partition (one log)                  └─ broker (one server)
     └─ segment (a file on disk)              └─ stores replicas of partitions
         └─ event, at an offset                    (one leader + followers per partition)     
```
### Topic and [[Partition]]
- **Topic**: like the name of a table, where ==every row (event) lives in some partition== and the **table name only groups them**. `orders` is a name; `orders` partitions 0, 1 and 2 hold the events. 
- **Partition**: ==a ordered, append-only log== with its own offsets, starting at 0.
	- Each event is stored in exactly one partition, chosen by its **key** (**same key → same partition**).
	- Order is guaranteed only within a partition. Because the key fixes the partition, all events for one key stay in order.
### Append-only log
> [!NOTE] To Note
> - Offsets only increase.
> - ==Events are never changed in place.== A change is a new event at the end.
> - Old events are removed only by the broker's cleanup (time/size retention, or compaction), never one at a time by a client.

This all belongs to a **single partition**
**08:15:02**, order 4217 is created:
```
offset  key         value
0       order_4217  {"status": "created", "amount": 60}
```
**08:16:10**, order 4217 is paid. The existing event is **not changed**. A new event is added at the end:
```
offset  key         value
0       order_4217  {"status": "created", "amount": 60}
1       order_4217  {"status": "paid",    "amount": 60}
```
**08:20:55**, order 4217 ships:
```
offset  key         value
0       order_4217  {"status": "created",   "amount": 60}
1       order_4217  {"status": "paid",      "amount": 60}
2       order_4217  {"status": "shipped",   "amount": 60}
```
### Event / Message
> [!NOTE] Event / Message
> One entry in a partition's log, identified by `topic + partition + offset`.

| Field              | Set by                | Purpose                                                                                            |
| ------------------ | --------------------- | -------------------------------------------------------------------------------------------------- |
| Headers (optional) | Producer              | Name–value attached to the metadata to describe the event                                          |
| Key (optional)     | Producer              | **Decides the partition an event goes to**; if `null`, the producer sends it to a random partition |
| Value              | Producer              | The payload (Data itself)                                                                          |
| Timestamp          | Producer (by default) | Event time                                                                                         |
| ==Offset==         | ==Broker, on append== | The **event's position within the partition**                                                      |
**How an event looks like**:
```text
topic:      orders
partition:  1                                ← hash("order_4217") mod 3
offset:     1                                ← assigned by the broker on append
timestamp:  1792311370000                    ← 2026-10-18 08:16:10 UTC
headers:    event_type = order_paid
            source     = checkout-service
            trace_id   = 9f2c41ab
key:        order_4217
value:      {"order_id": 4217, "customer_id": 88, "status": "paid", "amount": 60.0, "currency": "SGD"}
```
### Components
**Producer**: Sends events to brokers; each event lands in one partition, chosen by key.
**[[Consumer Group]]**: Set of consumers that **work together to read ==one topic==** as if they were a single reader. ==Each partition is read by exactly one consumer in the group==, so the partition count is the group's maximum parallelism.
- **Consumer**: Client that fetches events from brokers and ==tracks its own offset==.
**[[Kafka Cluster]]**: Set of brokers that act as one Kafka system. Production clusters usually have **at least 3, so each partition can have 3 replicas**.
- **Broker**: ==one== Kafka server; stores partition data on its disks and serves producers and consumers.






