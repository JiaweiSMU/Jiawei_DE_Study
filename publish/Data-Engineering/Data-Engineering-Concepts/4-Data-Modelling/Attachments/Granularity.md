**2 key rules**
- Declare the grain before choosing columns.
    - Every dimension key and measure in the fact table must have exactly one value per row at that grain. Otherwise the value is repeated across rows and `SUM` overcounts.
- Use ==one grain per table==.
    - Rows at different grains in one table make every `SUM` double count, so each grain gets its own physical table.
**Grain**
- What ==each row in a fact table represents==, stated in business terms (e.g. one row per visitor arrival). From this:
    - The **columns that uniquely identify a row**: `arrival_id`, or `visitor_id` plus the arrival timestamp if no arrival ID exists. `visitor_id` alone isn’t enough, since one visitor can arrive many times.
**Summary grain**
- A table built with ==`GROUP BY`== from the fact table, so many fact rows become one stored row. Its grain is always coarser than the atomic grain.
	- e.g. atomic grain: one row per spend transaction. Summary grain: one row per trip, built by grouping on `trip_id` and summing spend.
- Anything **grouped away (spend by category, day or merchant) can’t be recovered from it**. Non-additive measures must be stored as parts (`SUM` and `COUNT` instead of `AVG`).
### 3 Kinds of Grain
1. **Transaction: one row per event**
	- A ==new row is written whenever something happens==. Rows are **only added, never changed**.
		- e.g. each hotel booking adds one row. If no one books on a given day, no row is written for that day.
**2. Periodic snapshot: one row per thing per period**
 - At a ==fixed interval== (e.g. daily), the **state of something is recorded** (e.g. rooms occupied), whether or not anything happened. A row is written for every hotel every day, even on days with no bookings.![[Pasted image 20261017120903.png]]
   - Each row tells us where things stood at one point in time (e.g. end of the day).
**3. Accumulating snapshot: one row per process, filled in as it progresses**
- A ==single row== tracks something that ==moves through known steps==, with a ==date column for each step== (e.g. booked, check-in, check-out).
- The **row is created at the first step**, so all step columns exist from the start but the later ones are empty. They are **filled in as each step happens.**![[Pasted image 20261017121239.png]]
  - Used to **track a process from start to finish**. The steps must be **well-defined** in advance, because each one needs its own column.
  - Why one row: every step’s date is in the same row, so durations are a subtraction (lead time = check-in − booked). ==With one row per step, you’d have to join the table to itself==.
  - **Expensive to maintain because rows are updated**, not just added.