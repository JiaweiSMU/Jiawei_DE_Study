A cloud data warehouse (Snowflake, BigQuery, Redshift etc.) **serves the same purpose as a traditional [[Data Warehouse]], but separates compute from storage: 
- data lives in object storage (e.g. S3) as compressed columnar files 
- compute is a separate cluster that holds no permanent data and caches recently read blocks on local SSD.
### What the separation gives
- ==Independent scaling==: object storage grows without adding nodes, and compute holds no data, so each is sized on its own.
- ==Fast resizing==: there are no rows to redistribute. New compute starts empty and reads from the same object storage.
- Workload isolation: several compute clusters can read the same data, so loads do not compete with analysts' queries.
- Pay for use: compute is billed per second (or per byte scanned), not bought upfront for peak.

### What it costs
- **Cold reads are slower**: a block not in the local cache needs a network request to object storage.
