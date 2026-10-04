Split the list in half until each piece has one item, then merge the pieces back in order.

- Split: `[5, 2]` and `[8, 1]` → `[5] [2] [8] [1]`
- Merge pairs: `[2, 5]` and `[1, 8]`
- Merge those by repeatedly taking the smaller front item: 1, then 2, then 5, then 8 → `[1, 2, 5, 8]`

Halving gives log n levels and each level's merging touches all n items, so it's O(n log n). It's always O(n log n), whatever the input. The cost is O(n) extra memory for the merged copies.