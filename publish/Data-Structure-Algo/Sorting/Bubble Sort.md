Compare each neighbouring pair and swap them if they're in the wrong order. Repeat passes until nothing swaps.

- Pass 1: `[2, 5, 8, 1]` → `[2, 5, 1, 8]`. The largest value, 8, has "bubbled" to the end.
- Pass 2: `[2, 1, 5, 8]`
- Pass 3: `[1, 2, 5, 8]`

It's O(n²) because it makes up to n passes, each with up to n comparisons. It's simple, slow, and rarely used in practice.