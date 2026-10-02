- Fast
	- Optimized for many **reads & writes** (Changes, Insertions, Deletions etc.) at high concurrency
- Highly Concurrent (Support many **overlapping operations** at the same time)
- [[ACID Transactions]] Compliant
	- Atomicity: It either fully succeed or fail, it won't result in a partial completed state
	- Isolation: An individual transaction itself is invisible to others
- ==Data is [[Normalized]]==
	- So each updates only touches one row per fact (e.g. Placing an order can result in it touching multiple rows across tables such as `orders, order_items`) which makes the transactions short and lock scope small to allow for many transactions at the same time

 The data usually will be moved to [[OLAP]] systems to perform analytical work