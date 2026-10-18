Is a system that **stores events in ==append only logs==**, one per partition. 
- Producers write events to a **Topic**, and each event is **appended to one partition** (`Key`). 
- Consumers read from the partition at their own pace. 
- Designed for **many ==small events**== (1MB and below)
### Kafka Terminology
```text
Data (logical)                          Machines (physical)
topic                                   cluster
 └─ partition (a log)                    └─ broker (a server)
     └─ segment (a file on disk)              └─ stores replicas of partitions
         └─ record (an event),                     (one leader + followers
            at an offset                            per partition)
```
### Append only log, example:
> [!NOTE] To Note
> - Offset only increases !
> - ==No events are updated / deleted==

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
### Event
> [!NOTE] Event
> One entry in a partition’s log.
> Identified by: `topic + partition + offset`

An event contains:
- **Headers**: **Optional**, name-value pairs that **Producers** attach to describe the event (metadata). 
- **==Key** (Optional)==: Used to decide which **partition an event goes to**. So the same key always ends up in the same partition. If the key is `null` the event can go to any partition.
- **Value**: Payload (Data itself)
	- Limited to about **1MB** per batch by default
- **Timestamp**: Set by the `Producer`
- **Offset**: The **event's position within it's partition**. Set by the `Broker` when the event is appended.

**How an event looks like**
```text
topic:      orders
partition:  1                                ← hash("order_4217") mod 3
offset:     2                                ← assigned by the broker on append
timestamp:  1792311370000                    ← ms since 1970 = 2026-10-18 08:16:10 UTC
headers:    event_type = order_paid
            source     = checkout-service
            trace_id   = 9f2c41ab
key:        order_4217
value:      {"order_id": 4217, "customer_id": 88, "status": "paid", "amount": 60.0, "currency": "SGD"}
```
### Components
**Producer**: Sends events to Brokers
**[[Consumer Group]]**: Set of consumers that **work together to read ==one topic==** as if they were a single reader
- **Consumer**: Fetches events from Brokers
**[[Kafka Cluster]]**: Set of Brokers that together acts as one Kafka System, usually 3 brokers us the minimum
- **Broker**: ==One== Kafka Server that stores partition data on its disks and serves producers and consumers



Kafka Cluster: Several brokers working together
- `Broker`: Individual Kafka Server, where it stores the logs and sends data

How messages are organized:
`Topics`: Think of it like a Table. Then Producer writes to it and Consumers read from the table. Named stream of related events, that are written by a `Producer`. 
- `Partition`: Topics can be split into seveal partitions, and each partition is its own seperate log with its own offsets. This allows messages to be processed in parallel.
	```text
	orders, partition 0:  [0][1][2][3][4][5] ...  ← new events appended here
	orders, partition 1:  [0][1][2][3] ...
	orders, partition 2:  [0][1][2][3][4] ...
	```
	- Having partitions allow brokers to store different partitions and different consumers can read them in parallel.
	- Each partition holds different data. 
	- A topic is like the combination of all the partition, think of it like a table then within a table has rows, the topic is the table, partition is the rows
	- Order is only guranteed within a partition itself, outside of the partition there's no link for the order between different partition
`Producer` writes to a `Topic`.
`Consumer Group`: Group of consumers that reads from the same topic. But each consumer reads their own partition, so that no event is processed twice within the group.
- With 3 partitions, then maximum only 3 consumers.
- `Consumers` subscribe to a `Topic`



How messages are organized
- Topics
	- Partition: allows messages to be processed in parallel

Multiple consumers can read from a topic simultaneously

There's some kafka offset
- Used to track where the consumer has read only
- For continue when something fail

Retention policy to store the message 

It is scalable

Producer:
- If there's a key it will be sent to a specific partition else it will be sent to a random partition

consumer & consumer group
- consumer within a consumer group share responsibility to process things from different partition in parallel.
	- Usually one consumer is located to one partition
	- If anything happens to one consumer, another consumer will take over

Kafka cluster
- Usually multiple brokers
- They use some leader and follower model ?

