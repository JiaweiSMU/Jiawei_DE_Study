![[Pasted image 20261003224422.png]]
### Stages
#### **Bronze**
Stores the data `as-is`: no transformation, no business logic. Each record also carries ==load metadata== (load time, source file, run ID).

It is usually `append only` and `schema-on-read`.
- **Append only**: a load ==only adds records==. 
	- Why: the source keeps only its current state (rows are updated in place, an API returns the latest value). If each load overwrote bronze, earlier versions would be lost in both places.
- **Schema-on-read**: files are stored in the format the source delivered, and nothing is checked during the load. So that the ==ingestion can then fail only for transport reasons== (API down, permission denied), never because of the data's content.

| Load  | Source returns | Overwrite bronze holds | Append-only bronze holds                              |
| ----- | -------------- | ---------------------- | ----------------------------------------------------- |
| April | March = 100    | March = 100            | (March, 100, loaded April)                            |
| June  | March = 108    | March = 108            | (March, 100, loaded April), (March, 108, loaded June) |

**Replay**: rebuilding silver and gold by rerunning the transformation code over bronze, without going back to the source. The ==load time or run ID selects which loads to rerun.== 
Append only makes this possible because the input is:
- complete: every version the source ever delivered is still there
- fixed: a given load never changes, so the same code over the same load gives the same output

Cost: bronze grows without limit. Partition by load date and move old partitions to a cheaper storage class.
#### **Silver**
Contains `Cleaned & Transformed` data, the idea is to only apply ==just enough transformation==, so that it's usable for (Self Service Analytics)
#### **Gold**
Do more transformation so it's ready to be consumed by dashboards.
- It's usually [[Denormalized]] here and joins it via a [[Star Schema]] with `Facts & Dimensions`