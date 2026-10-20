An example:
```yaml
active_controller: 1001        # the controller currently making decisions

brokers:                       # every broker in the cluster
  - id: 1                      # the broker's number, used everywhere else in the metadata
    zone: ap-southeast-1a      # which availability zone it runs in
    alive: true                # still sending heartbeats to the controller
  - id: 2
    zone: ap-southeast-1b
    alive: true
  - id: 3
    zone: ap-southeast-1c
    alive: true

topics:
  - name: orders
    partitions:
      - partition: 0
        replicas: [1, 2, 3]    # brokers 1, 2 and 3 each hold a copy of this partition
        leader: 1              # broker 1 accepts the writes for this partition
        in_sync: [1, 2, 3]     # copies that are fully caught up with the leader

      - partition: 1
        replicas: [2, 3, 1]
        leader: 2
        in_sync: [2, 3, 1]

      - partition: 2
        replicas: [3, 1, 2]
        leader: 3
        in_sync: [3, 1, 2]
```
- **`replicas`**: the brokers holding a copy of this partition. The first is the preferred leader. It changes only when the partition is reassigned.
- **`leader`**: the broker whose replica leads this partition. It accepts writes, assigns offsets, tracks the high watermark, and serves reads by default.
- **`in_sync`**: the replicas, including the leader, that have kept up with the leader. The leader waits for these with `acks=all`, computes the high watermark from them, and a new leader is chosen from them.