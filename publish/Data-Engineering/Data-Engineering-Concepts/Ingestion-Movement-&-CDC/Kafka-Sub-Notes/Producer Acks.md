This `acks` is a producer side thing.
It's for the broker to acknowledge the producer

### 3 Kinds of Acks
1. `acks=0`: Producer doesn't wait for the broker, it's like a fire and forget.
	1. If the write fails, the event it lost
2. `acks=1`: Producer only waits for the leader to reply.
	```text
	producer ──send──> broker 1 (leader) writes event
	producer <──ack─── broker 1 replies right away
	                   (followers copy it a moment later)
	```
	1. Leader replies when it finishes writing the event
	2. The acknowledgement can arrive before any follow has a copy, if that happens the event is also gone and nothing reports an error
3. `acks=all`: Every in sync replica needs to receive the copy before a reply is sent back to the Producer
	```text
	producer ──send──> broker 1 (leader) writes event
	                   broker 2, broker 3 copy it
	producer <──ack─── broker 1 replies only once all in-sync replicas have it
	```
	1. Even if the leader dies, any replica that becomes the leader already has the event so nothing is list
	2. But this is slower

# Producer Acks
`acks` is a **producer** setting: ==how many brokers must have an event before the leader **acknowledges**== it to the producer.
### `acks=0`: fire and forget
- The producer doesn't wait for any reply.
- If the write fails, the event is lost and the producer never knows.
### `acks=1`: the leader only

```text
producer ──send──> broker 1 (leader) writes event
producer <──ack─── broker 1 replies right away
                   (followers copy it a moment later)
```
- The ==leader replies== as soon as it has ==written the event==.
- The ack can arrive **before any follower has a copy**. If the leader dies in that gap, a follower without the event becomes leader, and the event is lost with no error reported.
### `acks=all`: every in-sync replica

```text
producer ──send──> broker 1 (leader) writes event
                   broker 2, broker 3 copy it
producer <──ack─── broker 1 replies only once all in-sync replicas have it
```
- The leader ==replies only after every in-sync replica has the event.==
- If the leader dies, whichever replica becomes leader already has the event, so **nothing is lost.**
- **Slower**: the producer waits for the slowest in-sync follower.