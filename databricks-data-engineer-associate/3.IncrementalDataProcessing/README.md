# 3. Incremental Data Processing

## Structured Streaming

### Demo

```python
customers_df = spark.readStream
                      .format("json")
                      .schema(customers_schema)
                      .load("path/to/file")
```

### Write transformed data stream to Delta Table

\_Once the streaming query is executed, we need to stop it manually OR set the params to stop after several iterations.

```python
streaming_query = customers_transformed_df.writeStream
                          .format("delta")
                          .option("checkpointLocation", "<unique_location>")
                          .toTable("<location_to_store>")

streaming_query.stop()
```

### Modify the interval and output

\_To specify the iterations and output mode:

```python
customers_transformed_df.writeStream
                        .format("delta")
                        .outputMode("append")
                        .trigger(processingTime="2 minutes")
                        .option(...)
                        .toTable(...)
```

### Checkpointing

- Stores metadata about streaming query, execution plan
- Tracks processed offsets and committed results

When a batch starts, Spark first reads the `offset log` to get the **start** and **end** offset of the previous batch. It's the `write-ahead log` which is written at the beginning of the batch, so there is no guarantee that the data from the batch was successfully processed.

In order to find that, Spark then reads the `commit log`: if the batch id is in the `offset log` matches with the `commit log`, then that indicates that the previous batch succeeded. Spark starts reading from the end offset of the previous batch.

Otherwise, Spark rereads the data from the start offset of the previous batch. Additionally, Spark also offers once guarantees -> avoid duplication.

### Auto Loader
Auto Loader incrementally and efficiently processes new data files as they arrive in cloud storage without any additional setup. It provides a **Structured Streaming** source called `cloudFiles`.

AutoLoader reads files using **checkpoint**.

![alt text](./img/image-4.png)

\_To use AutoLoader for an easy ETL:

```python
spark.readStream.format("cloudFiles") \
  .option("cloudFiles.format", "json") \ # mandatory
  .option("cloudFiles.schemaLocation", checkpoint_path) \ # mandatory
  .option("cloudFiles.inferColumnTypes", "true") \
  .load("<path-to-source-data>") \
  .writeStream \
  .option("mergeSchema", "true") \
  .option("checkpointLocation", "<path-to-checkpoint>") \
  .start("<path_to_target")
```
 By default, the schema is inferred as `string` types (JSON, csv, xml), any parsing errors (there should be none if everything remains as a string) will go to `_rescued_data`, and any new columns will fail the stream and evolve the schema.

\_Use AutoLoader to load to a Unity Catalog managed table:
```python
checkpoint_path = "s3://dev-bucket/_checkpoint/dev_table"

(spark.readStream
  .format("cloudFiles")
  .option("cloudFiles.format", "json")
  .option("cloudFiles.schemaLocation", checkpoint_path)
  .load("s3://autoloader-source/json-data")
  .writeStream
  .option("checkpointLocation", checkpoint_path)
  .trigger(availableNow=True)
  .toTable("dev_catalog.dev_database.dev_table"))
```

#### Schema evolution
Auto Loader supports the following modes for schema evolution, which you set in the option `cloudFiles.schemaEvolutionMode`:
- `addNewColumns` (default): Stream fails. New columns are added to the schema. Existing columns do not evolve data types.
- `rescue`: Schema is never evolved and stream does not fail due to schema changes. All new columns are recorded in the `rescued data` column.
- `failOnNewColumns`: Stream fails. Stream does not restart unless the provided schema is updated, or the offending data file is removed.
- `none`: Does not evolve the schema, new columns are ignored, and data is not rescued unless the `rescuedDataColumn` option is set. Stream does not fail due to schema changes.

## Delta Lake

### Delta Transaction Log

![alt text](./img/image.png)

Let's say, first we created a `Delta Table` in `Unity Catalog`, and it also creates a folder in Cloud Storage to store the **file** and the **transaction log**.

Then, **Transaction 1 - Insert Data**, **Transaction 2 - Insert more data**, these transaction logs give Delta Lake the ability to offer ACID transactions as well as time travel capabilities.

![alt text](./img/image-1.png)

### Delta Lake Version History

\_To query the table history:

```sql
DESCRIBE HISTORY <table_name>
```

\_To query data from a specific version:

```sql
SELECT *
FROM ...
VERSION AS OF 1;

-- or use the timestamp
SELECT *
FROM ...
TIMESTAMP AS OF '<the_specific_timestamp>';

```

\_To restore the version:

```sql
RESTORE TABLE <table_name>
VERSION AS OF 1;

```
### Delta Lake ACID Transactions
- Firstly, the **Transaction Logs** only return at the end of the transaction. E.g., when you insert data into the table, data files will be return first and only AFTER successful completion of writes in the data files, the **Transaction logs** will be written -> transaction logs become the single source of truth.
- When reading the data files, Delta Lake will read the **Transaction logs**, therefore it will not read the corrupted files (since the **transaction logs** won't be created).

### Create - Drop table 
- `CREATE OR REPLACE` retains all the transaction history, whereas `DROP` and then `CREATE` the table removes all of that history. -> should use create/replace to retain the history. 

### Delta Lake Table & Column properties
_To see the comment of a table:
```sql
DESC EXTENDED <table_name>
```

_`TBL properties` (table properties): used to specify table level metadata or configuration settings.
```sql
CREATE TABLE <table_name>
(...)
TBLPROPERTIES ('sensitive' = 'true'); -- can also change table configs
```

_To create the **column properties**:
```sql
CREATE TABLE <table_name>
(company_id BIGINT NOT NULL GENERATED ALWAYS AS IDENTITY (START WITH 1 INCREMENT BY 1),
...)

```
When using the clause `GENERATED BY DEFAULT AS IDENTITY`, insert operations can specify values for the identity column. Modify the clause to be `GENERATED ALWAYS AS IDENTITY` to override the ability to manually set values.

_**CTAS (Create Table As statement)**: a single operation 
```sql
CREATE TABLE <table_name> AS
(
  SELECT *
  FROM ...
  WHERE ...
)
```
Drawbacks: you **CANNOT** specify the columns properties; if you want to define the column data types, then it must be done in the **SELECT** statement. OR, you need to alter the table after creation. 

```sql
ALTER TABLE <table_name>
ALTER COLUMN <col_name> TYPE BIGINT
```

### Insert Overwrite and Partitioning 
- **Insert Overwrite**: overwrites the **existing** data in a table or a specific partition with the new data. 
- **Insert Into**: appends new data 

_If we want to insert overwrite the partitions in different tables:
```sql
INSERT OVERWRITE TABLE <table_partition_name>
PARTITION (country="USA")
SELECT ...
FROM <table_from_usa>
```

> Handle schema changes 
<br> - **Insert Overwrite**: Use to overwrite the data in a table or a partition when there are no schema changes. 
<br> - **Create or replace** table: use when there are schema changes.

### Copy into and Merge 
_Copy into (incrementally load new files from Cloud Storage into Delta table). It can be considered as **lightweight auto loader**.
<br> So basically, if we already executed the first `COPY INTO` with a specific folder, then after that, we add a new file and execute the code once again, only the new file is read.


```sql
COPY INTO <table_name>
FROM 'path/to/file/or/folder'
FILEFORMAT = JSON
FORMAT_OPTIONS ('inferSchema' = 'true')
COPY_OPTIONS ('mergeSchema' = 'true') -- if a new column is found, just add into the table
```
<br>
_Merge into: merge the source data into target table

```sql
MERGE INTO <target_table> AS target
USING <source_data> AS source
ON <tgt.id = src.id>
WHEN MATCHED AND source.status ='ACTIVE' THEN
  UPDATE SET 
    target.price = source.price,
    target.trading_date = source.trading_date

WHEN MATCHED AND source.status = 'DELISTED' THEN 
  DELETE 

WHEN NOT MATCHED THEN
  INSERT (...) VALUES (...);

```

#### COPY INTO vs AUTO LOADER
- You want to load data from a file location that contains files in the order of **millions or higher**. `Auto Loader` can discover files more efficiently than the COPY INTO SQL command and can split file processing into multiple batches.

- You do not plan to **load subsets of previously uploaded files**. With Auto Loader, it can be more difficult to reprocess subsets of files. However, you can use the `COPY INTO` SQL command to reload subsets of files while an Auto Loader stream is simultaneously running.

### Delta Lake Compaction
_File compaction using `OPTIMIZE`: merge multiple small files into fewer, larger files. The history is still maintained. 
```sql
OPTIMIZE <table_name>
```

_File compaction using `ZORDER BY`: the data with specific z-order column values are kept together in the same file. -> optimize by searching through fewer data files.
```sql
OPTIMIZE <table_name>
ZORDER BY <column_name>
```

### Vacuum command 
- Used to removed, unused files from Delta Lake to free up storage. <br> E.g., when executing `OPTIMIZE`, the files are compacted into a larger file, leaving redundant smaller files.
- This command permanently removes data from the cloud storage, and it removes the files which are not referenced in the latest transaction log and older than the retention threshold (by default 7 days).

```sql
SET spark.databricks.delta.retentionDurationCheck.enabled = false; -- (optionally) disable the warning
VACUUM <table_name> RETAIN 1 HOURS; -- leave history for the last hour and delete files before that
```

### Deletion Vectors 
> Problem: you have millions of records in parquet file, and you want to delete only one single row -> all the file has to be rewritten 
<br> **Optimization solution:** enable table with `Deletion vector`.

How it works is similar to flagging: you flag the row as deleted, and then when the parquet file is read, that flagged row will be skipped.

Later on, when you run maintenance, (e.g., `OPTIMIZE` command or `REORG TABLE`), the parquet file will be rewritten and those marked rows will be removed. 

```sql
CREATE TABLE <table-name> [options] 
TBLPROPERTIES ('delta.enableDeletionVectors' = true);

ALTER TABLE <table-name> 
SET TBLPROPERTIES ('delta.enableDeletionVectors' = true);
```

### Liquid Clustering (in Delta tables)
This optimization is suitable when:
- Tables with significant skew in data distribution 
- Tables with access patterns that change over time 
- Tables where a typical partition column could leave the table with too many or too few partitions. 

To enable in SQL:
```sql
-- Create an empty Delta table
CREATE TABLE table1(col0 INT, col1 string) CLUSTER BY (col0);

-- Using a CTAS statement
CREATE EXTERNAL TABLE table2 CLUSTER BY (col0)  -- specify clustering after table name, not in subquery
LOCATION 'table_location'
AS SELECT * FROM table1;

-- Using a LIKE statement to copy configurations
CREATE TABLE table3 LIKE table1;
```

To enable in Python:
```python
# Create an empty Delta table
(DeltaTable.create()
  .tableName("table1")
  .addColumn("col0", dataType = "INT")
  .addColumn("col1", dataType = "STRING")
  .clusterBy("col0")
  .execute())

# Using a CTAS statement
df = spark.read.table("table1")
df.write.clusterBy("col0").saveAsTable("table2")

# CTAS using DataFrameWriterV2
df = spark.read.table("table1")
df.writeTo("table1").using("delta").clusterBy("col0").create()
```

---
To revert back to the simple table, you can:
```sql
ALTER TABLE <table_name>
CLUSTER BY NONE
```

#### Difference between Liquid Clustering vs Zorder:
- Liquid Clustering:
  - only reorganizes parts of the data that aren't already clustered to make it more efficient. 
  - ideal for scenarios with frequent updates
- Z-Ordering
  - reorganizes the entire table or partitions every time, which is more resource-intensive.
  - suited for read-heavy workloads.

## Delta Live Table
What is Delta Live Table?
- A **declarative** ETL framework for building reliable, maintainable, and testable data processing pipelines. 
- You define the transformation to perform on your data and Delta Live Tables manage task orchestration, cluster management, monitoring, data quality and error handling. 

![alt text](./img/image-2.png)

### Programming with DLT 
![alt text](./img/image-3.png)

```sql
CREATE OR REFRESH Live Dataset
[Data Quality Expectation]
AS 
  SELECT columns, transformations 
  FROM source  
```

3 types of Live Dataset: 
- `Streaming Tables`: Delta Table to which streams write data. Reads from EventHubs, Kafka, Cloud Files, ... Suitable for Incremental Data Ingestion; Allow DML operations; Massive amount of data  
- `Materialized Views`: Delta Table created from the result of a query. Suitable for full refresh data ingestion workloads. NO DML operations allowed  
- `Standard Views`: no physical storage of the data; scope limited to pipeline; cannot be published to Hive Metastore/Unity Catalog. Suitable for intermediate results to reduce complexity and enforce data quality constraints.  

### DLT Pipelines and Notebooks 
>Within a DLT notebook, it is NOT ALLOWED to mix different languages (e.g., you cannot have a cell in SQL then a cell in Python). So, **NO magic commands**.

_Create **streaming table** for streaming operation (note that we *cannot provide a schema* here as schema is configured in DLT pipeline when we create it). We can only provide 1 schema in the pipeline for all the notebooks that are attached to that pipeline.
```sql
CREATE OR REFRESH STREAMING TABLE <table_name>
COMMENT ''
TBLPROPERTIES('quality'='bronze')
AS 
  SELECT ...
  FROM cloud_files(
    'path/to/file/data',
    'json',
    map('cloudFiles.inferColumnTypes', 'true')
  );
```

Implementation using **Python** syntax: 
```python
import dlt

# create a DLT dataset
@dlt.table(
    name = 'new_bronze_address',
    comment = '',
    table_properties = {'quality': 'bronze'} # a dict to define
) 
def bronze_address():
  return(
      spark.readStream # indicates streaming table, otherwise spark.read()
              .format("cloudFiles")
              .option("cloudFiles.format", "csv")
              .option("cloudFiles.inferColumnTypes", "true")
              .load("/path/to/files/being/read")
  ) # return a dataframe
```
The name of the table will be the same as the function's name (i.e., `bronze_address`). In case you want different config, you can pass in additional args.

---
_Create a `MATERIAZLIZED VIEW` using Python:
```python
@dlt.table(
  table_properties = {"quality": "bronze"},
  commment = "customer bronze table",
  name = "customer_bronze"
)
def cust_bronze():
  return spark.read.table("dev.bronze.customer_raw")
``` 

---
_Create a `VIEW` (temporary) using Python:
```python
@dlt.view(  
  commment = "customer bronze table"
)
def joined_view():
  df_c = spark.read.table("LIVE.customer_bronze")
  df_o = spark.read.table("LIVE.order_bronze")
  
  return df_o.join(df_c, how="left_outer", on=df_c.id==df_o.id)
``` 

In case when you're at the silve table and want to choose from bronze table, use `LIVE` (in the same pipeline), and `STREAM` modifier helps with only reading the latest data since the last execution (as a stream rather reading whole data, to load data incrementally):
```sql 
...
  SELECT 
  FROM STREAM(LIVE.bronze_table)
```




**Reading data from a streaming Delta Table**:
```python
@dlt.table()

def silver_address_clean():
  return (
    spark.readStream.table("LIVE.bronze_address") 
  )
```

We can only execute that code block using `DLT pipeline`. Click `Workflow` on the left tab -> `Pipelines`-> `Create piplines` -> `ETL Pipeline`. The product edition also varies:
  - Core: can use dlt features
  - Pro: you can CDC (change data capture), but cannot do data quality
  - Advanced: everything

For **Compute option**. in **Cluster mode**:
- Enhanced autoscaling: optimize the cluster utilization by automatically allocating cluster resources based on your workload volume.
- Fixed size: the size of the cluster is fixed

In **Advanced option**, we config `pipelines.clusterShutdown.delay` to be 30m. 

Then, after having done with the configs, we start the pipeline:
- Default refresh mode: process the new data from an incremental source
- Full refresh mode: reset all the checkpoint, delete all the data from the streaming tables and reprocess all the available data.


_To add a new Notebook into the pipeline, go to `Setting` on top right, then `Add source code` -> choose the notebook.

---
#### DLT Pipeline mode
- `Development`: allows the dlt cluster to run after execution (The compute resources don’t terminate when the pipeline is stopped)
- `Production`: immediately kills the cluster after execution (either success/fail) (i.e., The compute resources terminate when the pipeline is stopped)

#### Pipeline result overview
![alt text](image.png)

> **Note:** All datasets in DLT are backed by Delta Tables and are managed by DLT Pipelines. So, if you delete a DLT pipeline, all datasets of that pipeline will be dropped. 
---

### DLT Expectations - Validating (only streaming tables and materialized views)
`Expectations` are applied for each of the record being processed. Expectation is basically just a conditional expression (e.g., id not null).

```sql
CONSTRAINT <expectation_name>
  EXPECT (<conditional_expression>)
  [ON VIOLATION (FAIL UPDATE | DROP ROW)]

-- e.g., 
CONSTRAINT valid_customer_id
EXPECT (customer_id IS NOT NULL)
ON VIOLATION FAIL UPDATE
```

To add the data quality rules:
```sql
CREATE OR REFRESH STREAMING TABLE silver_customer_clean(
  CONSTRAINT valid_customer_id EXPECT (customer_id IS NOT NULL) 
    ON VIOLATION FAIL UPDATE -- fail the data flow 
  CONSTRAINT valid_customer_name EXPECT (customer_name IS NOT NULL) 
    ON VIOLATION DROP ROW  
  CONSTRAINT valid_telephone EXPECT (LENGTH(telephone) >= 10) -- get a warning in case violation  
)
```

Using Python (only Python allows you to group multiple expectations using `expect_all`, `expect_all_or_drop`, and `expect_all_or_fail`):
```python
@dlt.expect("valid_customer_id", "customer_id IS NOT NULL")
@dlt.expect_or_drop("valid_customer_name", "customer_name IS NOT NULL")
@dlt.expect_or_fail("valid_telephone", "LENGTH(telephone) >= 10")

valid_records = {
                  "valid_customer_id": "customer_id IS NOT NULL",
                  "valid_customer_name": "customer_name IS NOT NULL",
                  "valid_telephone": "LENGTH(telephone) >= 10"
                }
@dlt.expect_all_or_drop(valid_records) # if any of them fail, the record will be dropped; also applies with similar decorators
```
> For more info, refer to Databricks docs on [data validataion](https://docs.databricks.com/aws/en/dlt/expectations?language=Python).

### Type 1 SCD 
It overwrites the data, which means that no history will be maintained.

**Apply changes API** - check in docs for extensive options.
```sql
APPLY CHANGES INTO <target_table>
FROM <src_table>
KEYS <columns>
SEQUENCE BY <columns>
STORED AS <SCD TYPE 1 | SCD TYPE 2>

APPLY CHANGES INTO LIVE.silver_customers
FROM STREAM(LIVE.silver_customers_clean)
KEYS(customer_id)
SEQUENCE BY created_date
STORED AS SCD TYPE 1;
```

### Type 2 SCD
The records will have **start_date** and **end_date**, where the new updated record will be added with a new **start_date**.
<br> -> We will have the full history of that record based on time.  

**Using Python syntax with `create_streaming_table()` and `apply_changes()`**:
```python
dlt.create_streaming_table(
    name = "silver_address",
    comment = "SCD type 2 address table",
    table_properties = {"quality": "silver"}
)

dlt.apply_changes(
  target = "silver_address",
  source = "silver_address_clean",
  keys = ["customer_id"],
  sequence_by = "created_date",
  stored_as_scd_type = 2
)
```


### Type 3 SCD
Only the most recent change is tracked.

## Lecgacy terminology
|     |           New terminology           |      Legacy terminology       |
| :-- | :--------------------------------:  | :---------------------------------: |
|     |          Materialized View          |           LIVE table           |
|     | CREATE OR REFRESH MATERIALIZED VIEW |  CREATE OR REFRESH LIVE TABLE   |
|     | Streaming table |  Streaming LIVE table   |
|     | CREATE OR REFRESH STREAMING TABLE   | CREATE OR REFRESH STREAMING LIVE TABLE | 
|


