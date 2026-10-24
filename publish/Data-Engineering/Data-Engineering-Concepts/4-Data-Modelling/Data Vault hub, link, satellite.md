Is one of the ways to build a data warehouse.
![[Pasted image 20261005222723.png]]
### Data Vault splits each entity into three kinds of table.
**Hub: which things exist**
- Holds the ==business key and nothing descriptive==, with one row per key.
- It is stable, because a ==business key rarely changes== (for example, an NRIC).
Satellite: what we know about each thing
- Holds the ==key plus the descriptive columns==. Foreign keys are excluded; they live in the link.
- Holds one row per version. Each **change inserts a new row with its own load timestamp**, and existing rows are **never updated**.
- Because rows are only ever ==inserted==, each with a timestamp and a source, the full history is kept. This is what gives ==traceability==.
**Link: which things are related**
- Holds the ==keys of the hubs it relates==, with one row per combination.
- It can connect two hubs or more (for example customer, order and store).
```sql
hub_customer          sat_customer
customer_no           customer_no | name    | segment
C-1001                C-1001      | Tan Wei | Retail

hub_order             sat_order
order_no              order_no | amount
O-501                 O-501    | 50

link_customer_order
customer_no | order_no
C-1001      | O-501
```
### Why it is flexible
Keys, descriptions and relationships sit in separate tables, so a change adds to the model without altering what already exists:
1. New column from an existing source: add the column to that source's satellite. The hub is untouched.
2. New source describing an existing entity: create a new satellite on the same hub.
3. New relationship: create a new link. If only the rule changes (an order can now have two customers), the existing link just gets more rows.
4. New entity: create a new hub with its own satellites.
### What it costs
The same split makes reads expensive. To get the ==current state of one customer, you join the hub to every one of its satellites and, in each satellite, keep only the latest row for that key==. The more satellites, the more joins. For this reason analysts do not query the vault directly; they query star schemas built on top of it.