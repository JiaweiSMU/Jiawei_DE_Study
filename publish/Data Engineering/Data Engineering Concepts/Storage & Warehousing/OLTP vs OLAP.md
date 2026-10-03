
|                         | OLTP                                                                    | OLAP                                                                                 |
| ----------------------- | ----------------------------------------------------------------------- | ------------------------------------------------------------------------------------ |
| **Workload**            | Many short transactions, each touching a few rows by key                | Few large queries, each aggregating millions of rows over a few columns              |
| **Typical query**       | `SELECT * FROM orders WHERE order_id = 4217`                            | `SELECT region, SUM(amount) ... GROUP BY region`                                     |
| **Latency target**      | Milliseconds                                                            | Seconds to minutes                                                                   |
| **Concurrency**         | Thousands of transactions at once                                       | Tens of queries at once                                                              |
| **Storage layout**      | **Row**: a page holds complete rows                                     | **Column**: each column stored separately in blocks                                  |
| **Cheap operation**     | ==Read or write==: about 5 pages, all columns in one page               | ==Aggregation==: reads only the columns used, heavily compressed                     |
| **Expensive operation** | Aggregation: every page carries all columns, so the whole table is read | Read or update: one block read and decompressed per column                           |
| **Modelling**           | **Normalized** (3NF): each fact stored once, so writes stay small       | **Dimensional** (star, snowflake): denormalized, since repeated values compress away |
| **Data**                | Current state, the system of record                                     | Historical copy, loaded by batch extract or change data capture                      |
