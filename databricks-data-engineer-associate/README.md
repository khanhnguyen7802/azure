# About

This is the guideline for the exam [Databricks Certified Data Engineer Associate](https://www.databricks.com/learn/certification/data-engineer-associate).

This `README.md` file will mostly reveal all necessary things to cope with the Databricks Certified DE Associate.

# Structure of the exam

1. Databricks Lakehouse Platform – 24%

   - Introduction to **Databricks Lakehouse** Platform
   - Databricks Workspace Components
   - Introduction to **Unity Catalog**

2. ELT With Spark SQL and Python – 29%

   - Overview
   - Querying data
   - Transforming data

3. Incremental Data Processing – 22%

   - Spark Structured Streaming
   - Delta Lake
   - DLT Overview
   - DLT - Pipelines and Notebooks

4. Production Pipelines – 16%

   - Databricks Jobs

5. Data Governance – 9%
   - Databricks SQL
   - Data governance


# Some additional notes post-revision

- `Delta log` is in json format and there is also crc file 
- `copy into` only works with sql
- `relational object`:
	+ table
		* external
		* managed
	+ view
		* standard: stored in metastore; persistent across sessions; resides in a specific schema/db
		* temporary: must be in the same notebook (same spark session); not stored in metastore
		* global: usually global temporary view -> so it's global compared to the temporary view 
	+ function
	
- fault tolerance: checkpointing + write-ahead logs 
- exactly once guarantee: idempotent sinks 
- delta live tables cannot use all-purpose cluster, only photon (serverless) or job cluster 
- constraint on violation drop row: violated records will be dropped and recorded as invalid in the event log
- `Cluster pools` (Compute -> Pool) help improve cluster start times by reusing idle VMs instead of provisioning new ones from scratch each time.

- Databricks SQL queries run slowly when submitted to a non-running SQL Warehouse
-> turn on the serverless feature to avoid the cold-start delay

- In Development mode, all datasets in DLT pipeline will be updated at set intervals until the pipeline is manually shut down. 
- By default, Auto Loader infers all columns as strings for file formats (e.g., json, csv) because they dont encode data types. 

- the `overwrite` mode first deletes all the files in Delta table's directory (but the old data is still available in the log history)
and then writes the new data to the table -> the table only reflects the latest data. 

- Delta operations are transactional and atomic. 
_In dlt pipeline, you can add the pipeline into DLT notebook to validate (debugging mode)

----
- coi lại control plane and data plane 
- worker node, driver node, dbfs are parts of the data plane, running in customer's cloud account 
- check cách xử lí table để vẫn giữ đc history
