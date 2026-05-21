# About

This is the guideline for the exam [Databricks Certified Data Engineer Associate](https://www.databricks.com/learn/certification/data-engineer-associate).

This `README.md` file will mostly reveal all necessary things to cope with the Databricks Certified DE Associate.

⚠️ Remember to check the official page to update the exam format changes. 

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
  - table
    - external
    - managed
  - view
    - standard: stored in metastore; persistent across sessions; resides in a specific schema/db
    - temporary: must be in the same notebook (same spark session); not stored in metastore
    - global: usually global temporary view -> so it's global compared to the temporary view
  - function
- fault tolerance: checkpointing + write-ahead logs
- exactly once guarantee: idempotent sinks
- delta live tables cannot use all-purpose cluster, only photon (serverless) or job cluster
- constraint on violation drop row: violated records will be dropped and recorded as invalid in the event log
- `Cluster pools` (Compute -> Pool) help improve cluster start times by reusing idle VMs instead of provisioning new ones from scratch each time.

- Databricks SQL queries run slowly when submitted to a non-running SQL Warehouse
  -> turn on the serverless feature to avoid the cold-start delay

- In Development mode, all datasets in DLT pipeline will be updated at set intervals until the pipeline is manually shut down.

- the `overwrite` mode first deletes all the files in Delta table's directory (but the old data is still available in the log history)
  and then writes the new data to the table -> the table only reflects the latest data.

- Delta operations are transactional and atomic.
  \_In dlt pipeline, you can add the pipeline into DLT notebook to validate (debugging mode)


- check cách xử lí table để vẫn giữ đc history

---
# Might be confusing 
## AutoLoader vs Spark Structured Streaming 
Autoloader is basically just Spark streaming under the hood with additional feature for event-driven ingestion.

If a job already starts and new files arrive in the source, then they will be queued for the next processing.

Autoloader is not 'real-time'; rather, it's considered a near-real-time process (500ms). It reads your files in micro-batches, which can be configured. The trigger is re-run every 500ms by default, unless set otherwise https://docs.databricks.com/en/structured-streaming/triggers

Benefits: https://docs.databricks.com/aws/en/ingestion/cloud-object-storage/auto-loader#benefits-of-auto-loader-over-using-structured-streaming-directly-on-files

## Delta Live Tables (DLT) vs Structured Streaming
> TL;DR: DLT = SaaS Structured Streaming, makes streaming simple to implement but with a cost ($$). It lets you  write your streaming code with fewer lines of code (though DLT still has other things to offer).

```py
# using Structured Streaming to read, write, clean then write back again
df_taxi_raw = spark.readStream.json('/databricks-datasets/nyctaxi/sample/json/')
df_taxi_raw.writeStream.format('delta').start('/path/to/delta/tables/taxi_raw')
df_filtered_data = spark.readStream.format("delta").load("/path/to/delta/tables/taxi_raw").where(...)
df_filtered_data.writeStream.format('delta').start('/path/to/delta/tables/filtered_data')

# while using DLT
import dlt

@dlt.view
def taxi_raw():
  return spark.read.format("json").load("/path/to/json/file/streams/taxi_raw")

@dlt.table(name="filtered_data")
def create_filtered_data():
  return dlt.read("taxi_raw").where(...)
```


## Delta Live Tables (DLT) vs Lakeflow Spark Declarative Pipelines (SDP)
Previously, Databricks used the `dlt` module to support pipeline functionality. The `dlt` module has been replaced by the `pyspark.pipelines` module. You may still use `dlt`, but Databricks recommends using `pipelines`. 

```py 
import dlt 
~ from pyspark import pipelines as dp
```

> Read more at: https://docs.databricks.com/aws/en/ldp/

