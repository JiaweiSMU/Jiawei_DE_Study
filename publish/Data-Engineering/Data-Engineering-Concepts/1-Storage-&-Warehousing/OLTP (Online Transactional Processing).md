Form of Data Processing where numerous **many short transactions run concurrently**. Each one reads or writes a few rows, located by key, and finishes in milliseconds.
### Characteristics 
- **Operations:** `inserts, updates, deletes, and selects` on a ==few rows by key==. Individual queries are simple; the hard part is running thousands of them at once.
* **Data is constantly modified,** unlike [[OLAP (Online Analytical Processing)]], where data is mostly appended in batches.
* [[Normalized]] (3NF): each fact is stored once, so changing a fact touches one row. ==Writes are small because the data is normalized==. Placing an order touches only a few rows across `orders` and `order_items`. **Small writes** keep transactions short and **locks limited to a few rows,** which lets **many transactions run at once**.
### Row layout
    Page 1: [1, SG, 50] [2, MY, 30] [3, SG, 20]
    Page 2: [4, ID, 70] [5, SG, 10] [6, MY, 40]
* A page (8 KB) holds many complete rows.
	*  To find a row, the engine will ==read the page to find it==.
	* [[Indexing]] is used to find the row faster else it will result in a sequential scan
* **Good for point operations by key:** one page read **returns every column of the row,** and an update writes one new row version in one page.
* **Bad for aggregation:** As every page carries all columns, so it would have to read every single page.
	* `SUM(amount)` must read the whole table to use one column. No column can be skipped.

 The data usually will be moved to [[OLAP (Online Analytical Processing)]] systems to perform analytical work
### Why analytics moves to OLAP
Analytical scans read the whole table and push the pages that transactions need out of memory, slowing the transactional workload. So the data is copied to an [[OLAP (Online Analytical Processing)]] system, by batch extract or change data capture, and analysed there.