# 4. Production Pipelines

## Databricks Workflows/Jobs
![alt text](./img/image.png)

Databricks Workflows is a **fully managed** orchestration service, fully integrated with the Databricks Data Intelligence Platform. 

It helps you 
- define a trigger for the pipeline based on the events
- create dependencies between the tasks so that they are executed in the required order
- also send notifications in the event of failure.

![alt text](./img/image-1.png)

Inside a specific **Job**, in **Job notification**, we can add `Metric threshold`.
- `Run duration` (duration metric): expected and maximum time for the run's completion
- `Streaming backlog (bytes)` [streaming metrics]: expected maximum bytes of unconsumed data across all streams. 
- `Streaming backlog (duration)` [streaming metrics]: expected maximum consumer delay across all streams. 
- `Streaming backlog (files)` [streaming metrics]: expected maximum number of outstanding files across all streams. 
- `Streaming backlog (records)` [streaming metrics]: expected maximum offset lag across all streams. 

## Introduction to Tasks
A job will have a number of different `tasks`. And each task will be in its own type and has its own properties. 

This procedure is similar to working with `DAGs in Airflow`.
![alt text](./img/image-2.png)

> A **job cluster** will do its job and after having finished, the cluster is terminated. And it's also less expensive than the **all purpose cluster**.

## Scheduling and Trigger 
> You can schedule in an **advanced mode** with `cron syntax`. Refer to [crontab generator](https://crontab.guru/) for more detail. 