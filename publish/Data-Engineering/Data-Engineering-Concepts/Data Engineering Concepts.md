### Storage and Warehousing
- [[OLTP vs OLAP]]
	- [[OLTP (Online Transactional Processing)]]
	- [[OLAP (Online Analytical Processing)]]
- [[3 V's of Big Data]]
- [[ACID Transactions]]
- [[Schema On Read]]
- [[Schema On Write]]
### Data Architecture![[Pasted image 20261003150123.png]]
- [[Diff between Data Warehouse, Data Lake and Data Lakehouse]]
	- [[Data Warehouse]] --> [[Data Mart]]
	- [[Cloud Data Warehouse]]
	- [[Data Lake]] --> [[Data Mart]]
	- [[Data Lakehouse]] --> [[Data Mart]]
### Open Table & File Formats
> [!NOTE] Append only tables vs Update tables
> Append-only suits events, which never change after they happen. Updatable tables suit things that have a current state.
- Open Table
	- [[Data Lake File Format]]
	- [[Lakehouse Table Format]]
		- [[Commit]]
		- Changing rows
			- [[Merge On Read]], [[Copy On Write]]
		- [[Schema Evolution]]
- File Formats
	- [[Parquet]]
	- [[Iceberg]]
	- [[Delta Lake]]
- [[Time Travel]]
### Data Architecture & Stacks
- [[Data Engineering Architecture]]
- [[Medallion Architecture]]
- [[Inmon vs Kimball]]
### Data Modelling
- [[Dimension Modelling]]: flourishes in the **Gold layer**, where performance, usability, and business semantics reign supreme.
- [[Data Vault hub, link, satellite]]: shines in the **Silver layer**, where schema evolution, historical traceability, and source system integration dominate.
- [[Normalized]]
- [[Fact & Dimension table]]
	- Fact (Quantitative): Numerical & Measurable
	- Dimensions (Descriptive): Categorical & Contextual
- [[Slowly Changing Dimensions]]
	- Type 1 - **Overwrite**: Updating email addr
	- Type 2 - **Add New Row**: Changing home address
- [[Granularity]]
- [[Cardinality]]
### Ingestion, Movement & CDC
- [[ETL (Extract, Transform, Load)]]
- [[ELT (Extract, Load, Transform)]]
- [[Reverse ETL]]
- [[Change Data Capture (CDC)]]
- [[Delta Load for Data Warehouse]]: Various approaches to load data from OLTP into the warehouse
	- [[Scan & Hash Table]]
	- [[Log-based (Write Ahead Logs)]]
	- [[Trigger-based]]
	- [[Watermark]]
- [[Kafka Cluster]]
	- [[Kafka]]
- [[Backfill]]