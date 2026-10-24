Pick a column and ask "if this value changes in the real world, how many rows do I have to update?" If the answer is one, that fact is normalized. If the answer is "however many rows happen to reference it", it isn't.
### Forms of Normalization
> [!NOTE] Explanation
> Key Column: This can be PK / FK `e.g.`: order_id, customer_id
> It's possible for a table to contain more than 1 key column.

Example: ![[Pasted image 20261005231919.png]]
#### 1 NF: 
==Each cell should only hold a single value== & Each row needs to be identifiable via a `key`![[Pasted image 20261005231929.png]]
- Violation: `orders(order_id, products)` where `products` holds `"A, B, C"`, or columns `product_1, product_2, product_3`.
- Fix: one row per order and product, in `order_items(order_id, product_id, qty)`
#### 2 NF
If a non-key column depends on only part of the key, move that column to a new table whose key is that part. 
**Every other column ==must depend on all the key columns==.** `qty` needs both the order and the product. 

For each column, ask ==what you need to know to look it up==:
- `qty`: the order and the product. It needs the whole key, so it stays.
- `product_name`: the product only. It moves out.
- `order_date`, `customer_id`, `customer_city`: the order only. They move out.
#### 3 NF
Other columns must also not depend on any non-key column![[Pasted image 20261005233811.png]]
- `discount_code` depends on the whole key (which code was used on this line), so it passes.
- `discount_percent` is decided by `discount_code`, a non-key column: SALE10 is always 10. That breaks 3NF, even though the table passes 2NF.
### Denormalization
Is to deliberately ==store a fact more than once==, undoing some of the steps of normalization, so that **queries need fewer joins**.
- ==Reads are faster==, because the columns a query needs are already together.
- Each fact is ==repeated on many rows==, so changing it means rewriting all of them, and copies can disagree if one is missed.
#### Common degrees of denormalization:
- **Star schema:** only the ==descriptive tables are flattened==. **Product, category and department** become one `dim_product` table. 
  The **central sales table** (Fact ==Stays in 3NF==) keeps just keys and numbers. Queries need one join per dimension.

