# 1. Implement and Manage an analytic solution
## Configure Microsoft Fabric workspace settings
### Configure Spark workspace settings
What you need to know: 
- Spark nodes
- Executors 
- Tasks 

![alt text](image.png)
- `Starter Spark tool`: 
  - constantly run in the background -> create automatically for a workspace (using **prehydrated live pools**) 
  - fast startup time 
  - consumes Fabric capacity when used
  - **auto scale** (maximum availabe nodes in the cluster) and **dynamically allocate executors** 
  - can be customized
- `Normal Spark pool`:
  - does not support CI/CD -> use **environment** to fix this issue
  - is tied to workspace
- `environment`: creates a new item to workspace 
  - one workspace can have multiple environments 
  - default evnironment can be defined for a workspace
  - provides **runtime version**.

  ![alt text](image-1.png)

- `high concurrency`: allow notebooks share a same Spark session 
  - better resource utilization
  - faster execution
  - can be used in data pipeline with **session tag** -> e.g., notebook A starts the cluster that takes a while, and after that, notebook B (which has the same session tag as A), can run immediately without waiting the cluster to start by using the same Spark session from notebook A (normally cluster spun up in A has already terminated as soon as the notebook has done its job).
  - Session sharing conditions:
    - same user runs the notebook 
    - same default Lakehouse
    - same Spark compute configuration
    - same libraries on environment level 
    - **up to 5 notebooks** can share the same session

- `Spark job setting`: 
  - Reserve maximum **cores** for active Spark jobs: Spark estimates the min and max number of cores for that job and reserved that -> *give more reliability but less optimal*

- Click into the created environment lets you customize `Spark compute configuration`. 


> Differentiate `session` vs `environment`: You configure an ML **environment** (Python 3.10, pandas 2.2, scipy 1.12, ...). This environment is saved in Fabric.
Then you open a notebook → Spark cluster spins up → a Spark **session** starts.\
Fabric provisions compute (= session).
The **session** loads the chosen **environment** (libraries, dependencies). Your code executes inside this session.

--
### Configure domain workspace settings
First off, what is a domain? \
-> a way of logically grouping together all the data in the organization that is relevant to a particular area or field -> a grouping of **workspaces**. 

Domains help business users with **data discoverability**.

![alt text](image-2.png)

*Read more at: https://learn.microsoft.com/en-us/fabric/governance/domains*

--
### Configure OneLake workspace settings
The OneLake File Explorer (tenant-level) makes data in OneLake accessible on your local machine.

![alt text](image-3.png)

> By default, PowerBI semantic models and KQL databases do NOT get written to OneLake. If you want to enable OneLake integration for KQL database/PowerBI semantic models, turn on the `Availability` button in OneLake and turn on the `OneLake Integration` in **Tenant-level setting** respectively.

--
### Configure data workflow workspace settings
`Apache Airflow Jobs (aka Data Workflows)` is orchestration tool. But Fabric provides a mechanism to run DAG using a **Pool**. 

Tbh, idk what the exam wants to test here. Maybe ingesting data with Dataflows gen 2 in Fabric?

## Implement lifecycle management in Fabric
### Configure version control
In Microsoft Fabric, version control means **connecting your workspace to Git** *(workspace level)*.

![alt text](image-4.png)


In `Workspace setting`, choose and connect to a Git provider (Github or Azure Devops).

Welp, this section is mainly about Github. Since I am familiar with Git/Github already, I'll skip this one. 

**Main things to take note:** changes, commit, push, merge, checkout branch. 

Requires Admin, Member or Contributor access to the source of workspace, and permission when branching out to a new workspace.

![alt text](image-5.png)

--
### Implement database projects
`SQL database project` is a local representation of SQL objects that comprise the schema for a single database, such as tables, stored procedures, or functions
-> The development cycle can be integrated into CI/CD deployment workflows.

The **development workflow** can be seen as follows: we clone the whole warehouse folder, then with help of extension "SQL Database Project" in VScode, we right click and choose `build`. The build only succeeds if there is no (syntax, logic, ...) error in the folder.  

![alt text](image-6.png)

> When deploying warehouse via Fabric deployment pipeline, there is no need to use and deal with database projects (the `dacpac`) since deployment pipeline will handle that in the background.

After having built the warehouse locally, you can `publish` by adding a new profile using the **SQL connection string** from the warehouse on Fabric. 


-- 

### Create and configure deployment pipelines
`Deployment pipeline` is a built-in tool in Fabric for managing content lifecycle. It can be used to deploy changes from one workspace/environment to another (kinda similar to Azure Data Factory). Can only grant an admin access to a deployment pipeline.

![alt text](image-7.png)

- Each block is called `stage` *(dev, test, prod)*.
  - A `stage` represents one step in the content lifecycle. 
  - Each stage is linked to different workspace. 
  - Min 2 and max 10. 
- `Item pairing`: Fabric matches items between stages (src and tgt) to know how specific item has changed. 
  - Items are paired based on their unique internal IDs -> can identify whether an item is renamed or that's a new item
  - Each item is given a status:
    - **Same as source:** No changes to the item detected  
    - **Different from source:** Item has changed in some way
    - **Only in source:** Item is new and has not been deployed
    - **Not in source:** Item doesn't exist in the source stage. Deployment will have no impact to these items.

  > With deployment pipelines, you can choose and deploy the selected items (deploy everything is not necessary).

  ![alt text](image-8.png)

- `Deployment rules`: let you change settings like data sources or parameters when moving content between stages. This helps make sure each stage uses the right setup WITHOUT manual changes. E.g., without deployment rules, the `test workspace` might read from the lakehouse in the `dev workspace`.

  ![alt text](image-9.png)
  
  > The table lists the type of items you can configure rules for, and the type of rule you can configure for each one. 
<hr>

## Configure security and governance

- Implement workspace-level access controls
- Implement item-level access controls
- Implement row-level, column-level, object-level, and folder/file-level access controls
- Implement dynamic data masking
- Apply sensitivity labels to items
- Endorse items
- Implement and use workspace logging

## Orchestrate processes

- Choose between a pipeline and a notebook
- Design and implement schedules and event-based triggers
- Implement orchestration patterns with notebooks and pipelines, including parameters and dynamic expressions