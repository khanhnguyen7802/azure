# Ingest and transform data (30–35%)

## Design and implement loading patterns

### Design and implement full and incremental data loads

For `incremental loading`, we can do it with either `watermark` column or using `CDC` _(change data capture)_. We will be using `Copy activity` to perform incremental loading.

![alt text](image-7.png)

> Take into account that different storages will have different mechanisms, therefore, be careful otherwise you'll do a **write append**, rather than write merge
> ![alt text](image-8.png)

> Another way to do `incremental loading` is to use `Copy activity + watermark value`. Watermark value is a column that helps track new or changed records (using timestamp or ID). \
> For more info, refer to: https://learn.microsoft.com/en-us/fabric/data-factory/tutorial-incremental-copy-data-warehouse-lakehouse

--

### Prepare data for loading into a dimensional model

`Data model`: acts as a blueprint, guiding how data is stored, retrieved and managed

`Fact` tables store measurable data and `Dimension` tables answer "Who? What? Where?"

`Star schema` and `Snowflake schema`:

![alt text](image.png)

`Slowly Changing Dimension (SCD)`:

- **Type 0**: No changes
- **Type 1**: Overwrite (old data is COMPLETELY replaced by new data, no history is kept)
- **Type 2**: Add a new reord + full history is tracked (typically use start/end date).
- Type 3: track previous value (store old value in another column) -> use when you only consider the last change (not full history)
  ![alt text](image-1.png)

--

### Design and implement a loading pattern for streaming data
An overview of streaming in Fabric: 

![alt text](image-9.png)


## Ingest and transform batch data

### Choose an appropriate data store

From source: https://learn.microsoft.com/en-us/fabric/fundamentals/decision-guide-data-store

![alt text](image-2.png)

--

### Choose between dataflows, notebooks, KQL, and T-SQL for data transformation

![alt text](image-3.png)

--

### Create and manage shortcuts to data

Use `shortcuts` to quickly pull data from internal and external locations into your lakehouses, warehouses or datasets. `Shortcuts` can be updated or removed from your item, but these changes will not affect the original data and its source.

To create shortcuts: `Home` -> `Get data` -> `New shortcut`

![alt text](image-4.png)

> Some rules about shortcut caching:

![alt text](image-5.png)

--

### Implement mirroring

`Database Mirroring`: is a Fabric feature, which allows you to **"Mirror"** a supported database into Fabric, in near-real-time, without the need for ETL (no need for complicated pipeline). After that, you can use Fabric tools directly on mirrored data.

![alt text](image-6.png)

3 types of Mirroring:

- `Database Mirroring`

  - replicate databases and tables into OneLake
  - mirrored data is written to Delta tables
  - keep data fresh with near real-time sync
  - mirror is readonly
  - changes in the source are automatically reflected in the mirror
  - table data, schema and basic metadata gets mirrored
  - currently supported for: Cosmos DB, SQL DB, Snowflake, SQL managed instance, Azure database for PostgreSQL flexible server

- `Metadata Mirroring` _(in Azure Databricks)_

  - syncs only metadata and uses shortcuts to access the data
  - only mirrors metadata, without copying the actual data
  - is made possible by having data in Azure Data Blob Storage in Delta parquet format already
  - use shortcuts to point to the data, but uses metatdata to display data in more easily usable way
  - source datastore metadata and schema get mirrored

- `Open mirroring`
  - enables any application to write change data directly into a mirrored database in Fabric.
  - allow you to mirror data from any data store or souce system
  - use CDC to track insert, delete and update
  - CDC can be written as CSV or Parquet
  - works as database mirroring but CDC logic is not provided out of the box

A mirrored database can be created in `Workspace`, simply click `"New item"` and then search for `Mirror`.

--

- Ingest data by using pipelines
- Transform data by using PySpark, SQL, and KQL
- Denormalize data
- Group and aggregate data
- Handle duplicate, missing, and late-arriving data

## Ingest and transform streaming data

- Choose an appropriate streaming engine
- Choose between native storage, mirrored storage, or shortcuts in Real-Time Intelligence

-- 
### Process data by using eventstreams
![alt text](image-11.png)

There are different types of sources used in `Eventstream`.
![alt text](image-12.png)

`Evenstream Data Transformation`:

![alt text](image-13.png)


--
- Process data by using Spark structured streaming
- Process data by using KQL

--
### Create windowing functions
`Groupby` and `Window` function are used to group events over time. 
- `Tumbling window`
  - Fixed-size, non-overlapping, contiguous time intervals. 
  - Each event belongs to exactly one tumbling window. 
  - At the end of each window, you emit an aggregate (count, sum, avg, etc.).
  - **Use case:** Suppose you have sensor readings from IoT devices reporting temperature every second. You want to compute the average temperature every 1 minute.


- `Snapshot window` *(less common in Fabric scenario)*
- `Hopping window`
  - Windows of fixed length, but **overlap**, because each window “hops” forward by a hop size that can be smaller than the window size. 
  - Events can belong to **multiple windows**. 
  - You specify three parameters: time unit, window size, and hop size (optionally an offset).
  - **Use case:** Imagine you are processing click-stream data from a website, and you want to compute the number of clicks in the *last 5 minutes*, but update that count *every 1 minute*. \
  Here, each window is 5 minutes long, but because of the 1-minute hop, you'll get overlapping windows, so the count will reflect the “rolling” clicks.

- `Sliding window`
  - considers all possible windows of the specified length, but in practice, it only emits when the **content changes** (i.e. when an event enters or exits the window).
  - dynamic: you don’t wait for a fixed schedule; the window result updates whenever the window’s membership changes.
  - **Use case:** Suppose you're monitoring login attempts to a system for anomaly detection. You want to detect if there are **more than 10 login attempts in any 2-minute sliding interval**. \
  This will emit a result as soon as the count in any 2-minute sliding interval crosses the threshold, not just at fixed boundaries.

- `Session window`
  - group events based on “activity session,” not fixed time intervals.
  - **use case:** Imagine you're tracking user activity on a website, and you want to compute **how long each “session” lasts**. A session is defined as a series of user actions, but if there’s more than 5 minutes of inactivity, the session is considered ended. Assume also no session should exceed 1 hour. \
  If a user doesn’t produce any event for **5 minutes**, the window “closes” and a session record is emitted. But if the session keeps being active, it will keep extending until it hits **60 minutes**, then closes. 

![alt text](image-10.png)

```
Summary:
- Tumbling → periodic summaries
- Hopping → rolling metrics with overlap
- Sliding → continuous detection / anomaly when content changes
- Session → user or device activity sessions

```