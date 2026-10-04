
| Algorithm | Average    | Worst      | Extra space | One-line idea                                                |
| --------- | ---------- | ---------- | ----------- | ------------------------------------------------------------ |
| Bubble    | O(n²)      | O(n²)      | O(1)        | Swap adjacent pairs repeatedly                               |
| Insertion | O(n²)      | O(n²)      | O(1)        | Insert each item into the sorted part; O(n) if nearly sorted |
| Merge     | O(n log n) | O(n log n) | O(n)        | Split in half, sort each, merge                              |
| Quick     | O(n log n) | O(n²)      | O(log n)    | Pick a pivot, partition smaller and larger, recurse          |
