This is how you sort playing cards in your hand: take the next card and slide it into the right place among the ones already sorted.

- `[5 | 2, 8, 1]`: take 2 and put it before 5 → `[2, 5 | 8, 1]`
- Take 8, which is already in place → `[2, 5, 8 | 1]`
- Take 1 and slide it to the front → `[1, 2, 5, 8]`

The worst case is O(n²), because each item may slide past all the others. If the list is already nearly sorted, almost nothing slides, so it's close to O(n).