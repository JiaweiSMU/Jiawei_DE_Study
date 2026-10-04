A table format is a ==specification for **metadata** that records a table's schema and exactly which data files make up each version==. Each write produces **one new version, and the catalog points to the current one**. 
- As covered in [[Data Lakehouse]]
### Downsides
- Frequent small writes (e.g. a job committing every minute) create many small data files, and each commit also adds a snapshot and a manifest.
	- Query planning slows down because there is more metadata to read.
	- Compaction is needed to rewrite small data files and manifests into larger ones.
- Old snapshots keep replaced files in storage.
	- Snapshots must be expired before those files can be deleted.
- Compaction and snapshot expiry do not run by themselves. They are jobs I must schedule.

