### Fact Table
- Records the events of one business process, ==one row per event== at the same **grain**.
- Each row has **two kinds** of columns:
    - **Keys** that point to dimensions: who bought, what, where, when. The descriptions behind those keys.
    - **Measures**: the numbers you add up, such as quantity and amount.
### Dimension Table
- **Describes the things involved in business events**: who, what, where, when. It has one row per thing: one customer, one product, one store, one day.
	- **descriptive attributes** (name, category, region) that you ==filter and group by==.
- One dimension is **shared** by every fact table that needs it. 