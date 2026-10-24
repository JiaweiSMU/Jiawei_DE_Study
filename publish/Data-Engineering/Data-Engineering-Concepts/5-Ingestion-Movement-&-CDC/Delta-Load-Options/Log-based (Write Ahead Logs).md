The WAL is the ==database’s log of every change==, in order. Postgres writes it so ==committed changes survive a crash==, and reuses it for replicas and for log-based CDC.
- `INSERT, UPDATE & DELETE` are written into the WAL log
### When CDC reads it
See [[ACID Transactions]] for the transaction and WAL steps.
- CDC reads the WAL continuously, so it can see a transaction’s row changes while the transaction is still running.
- It holds those changes back, grouped by transaction, until it ==reaches that transaction’s commit record in the WAL (flushed to disk)==. It then sends all of the transaction’s changes downstream together.
- If the transaction aborts instead, CDC discards its changes. Only committed changes ever reach the pipeline.
### Log-based CDC on AWS
CDC tools such as (**AWS DMS, Debezium**), connects to Postgres and ==get the WAL row changes== and will ==stream== them to some places
- A **Replication slot**, is used to ==record how far the tool has read until==. So that in the event of a crash, the tool can resume from there and nothing is lost.