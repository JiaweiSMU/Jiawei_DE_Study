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