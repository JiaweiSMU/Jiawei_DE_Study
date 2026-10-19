A group of servers that work together and appear as one Kafka system. It has two kinds of server:
- **Brokers** ==store partition replicas== and serve producers and consumers.
- **Controllers** keep the cluster’s **metadata**. They store no events.
Example: 3 controllers, 3 brokers, two topics 
- `orders` with 3 partitions and replication factor 3
- `payments` with 4 partitions and replication factor 2.
```text
Kafka cluster
├── Controllers                        manage the cluster's metadata
│   ├── Controller 1   (active)        makes decisions, writes them to the metadata log
│   ├── Controller 2   (standby)       holds a copy of the metadata log
│   └── Controller 3   (standby)       holds a copy of the metadata log
│
├── Broker 1                           6 replicas, leader for 3
│   ├── orders   p0    LEADER
│   ├── orders   p1    follower
│   ├── orders   p2    follower
│   ├── payments p0    LEADER
│   ├── payments p2    follower
│   └── payments p3    LEADER
│
├── Broker 2                           5 replicas, leader for 2
│   ├── orders   p0    follower
│   ├── orders   p1    LEADER
│   ├── orders   p2    follower
│   ├── payments p0    follower
│   └── payments p1    LEADER
│
└── Broker 3                           6 replicas, leader for 2
    ├── orders   p0    follower
    ├── orders   p1    follower
    ├── orders   p2    LEADER
    ├── payments p1    follower
    ├── payments p2    LEADER
    └── payments p3    follower
```
### Partition placement
> [!NOTE] Partitions
> A topic can have **more partitions than there are brokers**, because one broker can hold replicas of several partitions. 
> The **replication factor** (replicas per partition), however, can be ==**at most the number of brokers**==, because no broker holds two replicas of the same partition.
- **Replica**: ==each copy of a partition is a replica==, stored on a broker. **No broker holds two replicas of the same partition.**
- **Replication factor:** the **number of replicas of each partition**. It is set **per topic**, and it can be **at most the number of brokers**. 
	- `orders`: 3, so every broker holds every `orders` partition. `payments`: 2, so each `payments` partition is on only two brokers.
- **Leader and follower:** ==roles **per partition**==, not per broker. For each partition, **one replica is the leader and the others are followers**. A broker is therefore the **leader for some partitions and a follower for others**. Leadership is spread across brokers so the ==write load is shared==.

### Controllers and metadata
[[Metadata]] is the cluster’s record of its own layout: which **brokers are alive, which topics and partitions exist, where each partition’s replicas are, and which replica leads**.
- **Controllers** keep the metadata as a log replicated among them. **One is active**; the others hold copies, ready to take over. A change is committed once a **majority** of controllers have it (log).
- **Brokers** send the active controller **==heartbeats==**. A broker that stops is declared dead.
- The **active controller** makes cluster-level decisions, mainly choosing a **new leader** when a leader’s broker dies, and writes them to the metadata log.
- **Brokers continuously fetch** that log from the active controller, so each knows the current layout and its own duties.
- **Clients** ask **any broker** for the metadata to find each partition’s leader.
### How an event is written
==Producers **always** send an event to the **leader** of its partition==. The leader **appends the event and assigns its offset**. Followers **fetch** from the leader and append the same events in the **same order**, so every replica matches the leader’s log, though a follower may be slightly behind.
- Only the leader writes because if several replicas accepted writes, **two of them could give different events the same offset**, and the copies would no longer match.
#### When an event is safe
**Rule:** an **event** counts as ==written only once **every in-sync replica** has the event==. Until then, it exists only on the leader and can disappear if the leader crashes. Kafka enforces this on both sides:

| Side         | Mechanism                                                                                   | Effect                                                                                                                                                                  |
| ------------ | ------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Producer     | **`acks=all`**: the leader confirms the write only once every in-sync replica has the event | If the leader crashes first, the producer gets no confirmation and resends to the new leader. With `acks=1`, the leader confirms immediately, so the event can be lost. |
| ==Consumer== | ==**High watermark**==: the highest offset every in-sync replica has, tracked by the leader | Consumers can read only up to it, so they never read an event that might disappear                                                                                      |
_Example:_ the leader (broker 1) has written an event at offset 5, but the followers have only up to offset 4. The high watermark is 4, so consumers can read only up to offset 4.

_Without the high watermark:_ a consumer reads offset 5, then the **leader** crashes before the followers copy it. A follower becomes the new leader without that event and assigns offset 5 to the next incoming event. Then:
- the **consumer holds an event that no longer exists in Kafka**;
- its bookmark points to 6, so it skips the new event at offset 5;
- other consumers never see the original event, so **consumers disagree about what happened**.
### Where consumers read from
By default, consumers **read from the leader**. Consumers can be configured to read from a follower by following the high watermark set by the leader, the trade-off is that **new events appear slightly slower**.






