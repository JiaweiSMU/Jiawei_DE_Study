In **OLTP** source, the `customers` table keeps one row per customer and overwrites that row when an **attribute changes**
```sql
-- Once this commits, no query can read the old value
UPDATE customers SET region = 'South' WHERE customer_id = 881;
```
Order row doesn’t store the region. It stores `customer_id`, and the region is read from the customer row through a join when the query runs. 

**Situation**
Customer 881 placed orders in 2025 while living in North, then moved to South in 2026. When we sum 2025 sales by region, there are two valid answers:
- ==**As it was==:** the orders count under North, the customer’s region when each order was placed.
- ==**As it is==:** the orders count under South, the customer’s current region.
After the overwrite, only the as-is answer can be computed. The value North no longer exists anywhere a query can reach.
### Type 1 - Overwrite
The warehouse applies the same overwrite to its dimension table:
```sql
UPDATE dim_customer SET region = 'South' WHERE customer_id = 881;
```
The dimension keeps ==one row per customer==, and that row ==always holds the current value==. No fact rows change.
Past orders join to that row by key when a query runs, so ==every past order now reports the new value==. As a result, **reports about the past change when a customer’s attribute changes**:
- The 2025 sales-by-region query run in 2026 counts 881’s orders under North.
**When to use it:**
- **Corrections**, such as a misspelled name. The old value was never a true state, so there is no history to keep.
- Attributes where only the **current value is ever wanted.**
### Type 2 - New row per version
> [!NOTE] Surrogate Key
> Internal key that is created to make a record unique but has no specific business value. 
> - Generally is created by hashing all the columns and use the hash as a SK

A Type 2 dimension keeps one row per version of each customer. These columns make it work:
- `valid_from`: Start of the version
- `valid_to`: End of the version
- `is_current`: True only on the latest version
- `surrogate_key`: Key generated **for each dimensional row**, it's what makes the ==row unique per version==. Since `customer_id` can repeat and can no longer identify a row
![[Pasted image 20261017104730.png]]
**On a change**, the load makes two writes in one transaction:
1. ==Close the current version==: set `valid_to` to the change time and `is_current` to false.
2. ==Insert the new version== with a new `customer_sk`, `valid_from` equal to that same change time, `valid_to` = `9999-12-31`, and `is_current` = true.