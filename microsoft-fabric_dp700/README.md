# About

This is the guideline for the exam [Microsoft Certified: Fabric Data Engineer Associate](https://learn.microsoft.com/en-us/credentials/certifications/fabric-data-engineer-associate/?practice-assessment-type=certification).

This README.md file will mostly reveal all necessary things to cope with the exam.

# Structure of the exam

## Skills measured as of April 21, 2025

As a candidate for this exam, you should have subject matter expertise with data loading patterns, data architectures, and orchestration processes. \
Your responsibilities for this role include:

- Ingesting and transforming data.
- Securing and managing an analytics solution.
- Monitoring and optimizing an analytics solution.

You should be skilled at manipulating and transforming data by using Structured Query Language (SQL), PySpark, and Kusto Query Language (KQL).

## Skills at a glance

### Implement and manage an analytics solution (30–35%)

#### Configure Microsoft Fabric workspace settings

- Configure Spark workspace settings
- Configure domain workspace settings
- Configure OneLake workspace settings
- Configure data workflow workspace settings

#### Implement lifecycle management in Fabric

- Configure version control
- Implement database projects
- Create and configure deployment pipelines

#### Configure security and governance

- Implement workspace-level access controls
- Implement item-level access controls
- Implement row-level, column-level, object-level, and folder/file-level access controls
- Implement dynamic data masking
- Apply sensitivity labels to items
- Endorse items
- Implement and use workspace logging

#### Orchestrate processes

- Choose between a pipeline and a notebook
- Design and implement schedules and event-based triggers
- Implement orchestration patterns with notebooks and pipelines, including parameters and dynamic expressions

### Ingest and transform data (30–35%)

#### Design and implement loading patterns

- Design and implement full and incremental data loads
- Prepare data for loading into a dimensional model
- Design and implement a loading pattern for streaming data

#### Ingest and transform batch data

- Choose an appropriate data store
- Choose between dataflows, notebooks, KQL, and T-SQL for data transformation
- Create and manage shortcuts to data
- Implement mirroring
- Ingest data by using pipelines
- Transform data by using PySpark, SQL, and KQL
- Denormalize data
- Group and aggregate data
- Handle duplicate, missing, and late-arriving data

#### Ingest and transform streaming data

- Choose an appropriate streaming engine
- Choose between native storage, mirrored storage, or shortcuts in Real-Time Intelligence
- Process data by using eventstreams
- Process data by using Spark structured streaming
- Process data by using KQL
- Create windowing functions

### Monitor and optimize an analytics solution (30–35%)

#### Monitor Fabric items

- Monitor data ingestion
- Monitor data transformation
- Monitor semantic model refresh
- Configure alerts

#### Identify and resolve errors

- Identify and resolve pipeline errors
- Identify and resolve dataflow errors
- Identify and resolve notebook errors
- Identify and resolve eventhouse errors
- Identify and resolve eventstream errors
- Identify and resolve T-SQL errors

#### Optimize performance

- Optimize a lakehouse table
- Optimize a pipeline
- Optimize a data warehouse
- Optimize eventstreams and eventhouses
- Optimize Spark performance
- Optimize query performance

## Some reviews on Reddit about the exam

### 1.

First Attempt: 540 – Relied too much on practice tests, lacked depth.
Second Attempt: 912 – Big change after:

Re-read MS Learn modules thoroughly.

Practiced SQL, KQL, PySpark daily.

Watched Aleksi’s videos again for better clarity.

Did Fabric hands-on labs to get real experience.
Actual Exam Experience
Total Questions: 54

Exam Pattern:

Part 1 – Case Study (10 Qs, can’t go back later, spend only 20 mins here)

Part 2 – 44 Questions ranging from easy → medium → difficult.

Tips:

Use MS Learn for last-min reference if you know exactly where to look.

Manage time carefully: Don’t spend too long on similar looking answers or on MS learn.

Case Study answers require quick comprehension.

### 2.

Regarding what resources I used to study:

I studied mostly watching youtube videos. shoutout to Aleksi Partanen and Will. I watched both series and I find that it was worth my time to watch both. They both cover all the topics but explain things differently and in different order. Some topics were easier for me to understand on one compared to the other.

I did have a Fabric Environment but to be honest I did not follow along on the videos. It feels repetitive to me. I just did basic navigation and creation of objects and perhaps real time intelligence since I was not very familiar with.

What I did find useful for practicing was the Microsoft learn modules. I left this at the end and didn't even have time to finish it. Most people disregard it as not covering enough ground which might be true but it's great for practice and they do mention stuff you don't see elsewhere.

Always relied on the documentation for topics I felt there was more to it i.e. spark settings.

I did the official practice assessments and certiace practice questions. This helped me see what knowledge I was still lacking but don´t expect to have similar questions in the exam.

Some advice:

Time management is important, I used Will's advice on allocating time between case study, regular questions, and reviewing answers.

Use the Sandbox, current practice assessments do not reflect the different type of questions you´ll get.

There are very specific syntax/code related questions, although not many. The level of detail surprised me. MS Learn helped me in some cases.

MS Learn helped me with a few questions. You will not get an index like the current documentation has. I would use the search bar to get to one topic and that topic in some cases had the link to the thing I was looking for. In some cases the search bar didn't return anything useful at all. I only started using while I was reviewing answers.

### 3.

Too many T-SQL questions

# Microsoft Fabric Structure

![alt text](image.png)

# Last moment revision

Within an eventhouse, you can create:

- KQL databases: Real-time optimized data stores that host a collection of tables, stored functions, materialized views, shortcuts and data streams.
- KQL querysets: Collections of KQL queries that you can use to work with data in KQL database tables. A KQL queryset supports queries written using Kusto Query Language (KQL) or a subset of the Transact-SQL language.

`Activator` is a technology in Microsoft Fabric that enables automated processing of events that trigger actions.

> For example, you can use Activator to notify you by email when a value in an Eventstream deviates from a specific range or to run a notebook to perform some Spark-based data processing logic when a Real-Time Dashboard is updated.

KEY CONCEPTS:

- Events: Each record in a stream of data represents an event that has occurred at a specific point in time.
- Objects - The data in an event record can be used to represent an object, such as a sales order, a sensor, or some other business entity.
- Properties - The fields in the event data can be mapped to properties of the business object, representing some aspect of its state.
  > For example, a total_amount field might represent a sales order total, or a temperature field might represent the temperature measured by an environmental sensor.
- Rules - The key to using Activator to automate actions based on events is to define rules that set conditions under which an action is triggered based on the property values of objects referenced in events.
  > For example, you might define a rule that sends an email to a maintenance manager if the temperature measured by a sensor exceeds a specific threshold.

---

`Eventstream`:
You can stream data from Microsoft sources and also ingest data from non-Microsoft platforms including:

- Microsoft sources, like Azure Event Hubs, Azure IoT Hubs, Azure Service Bus, Change Data Capture (CDC) feeds in database services, and others.
- Azure events, like Azure Blob Storage events.
- Fabric events, such as changes to items in a Fabric workspace, data changes in OneLake data stores, and events associated with Fabric jobs.
- External sources, such as Apache Kafka, Google Cloud Pub/Sub, and MQTT (Message Queuing Telemetry Transport)

---

What you can `monitor`:

- data pipeline activity
- dataflows
- semantic model refreshes
- Spark jobs, notebooks and lakehouses
- Microsoft Fabric Eventstreams

---

You can use `dynamic management views` (DMVs) to retrieve information about the current state of the data warehouse. Specifically, Microsoft Fabric data warehouses include the following DMVs:

- sys.dm_exec_connections: Returns information about data warehouse connections.
- sys.dm_exec_sessions: Returns information about authenticated sessions, showing session info like login time, client details, and resource usage.
- sys.dm_exec_requests: Returns information about active requests.

---

What is `OneLake availability` for?

Its primary purpose is **Cross-Engine Interoperability**.
In Microsoft Fabric, the Data Warehouse uses a highly optimized SQL engine.
While it stores data in OneLake in the Delta Parquet format, that data isn't automatically "visible" to other engines (like Spark or KQL) in a user-friendly way unless OneLake availability is active.
It is used for:

- Allowing Spark to read Warehouse tables: Enabling data engineers to use Python/Notebooks to query data residing in a SQL Warehouse without moving it.
- Simplifying Data Integration: Making Warehouse tables appear like standard Delta tables that can be "shortcutted" into Lakehouses.

-> When you enable OneLake availability, you are essentially telling Fabric to synchronize that proprietary data into the OneLake Delta format.

---

**Data pipeline vs Dataflow gen 2**

- Copy activity in a Data pipeline is specifically designed for high-performance data movement (optimized for bulk data movement)
- Dataflow Gen2 is primarily for transformation, a "transformation" tool for users who prefer low-code interface.
- Dataflow Gen2 processes data through the Power Query Mashup engine, which introduces more overhead.
  > For a straight "copy" from SQL to Warehouse, the Data Pipeline engine is significantly faster and more scalable.

-> Choose Data Pipeline if you see: "Bulk copy," "High performance," "Orchestration," "No transformation," or "Migration." \
-> Choose Dataflow Gen2 if you see: "Low-code," "Power Query," "Business user," "Complex cleaning," or "Merging multiple sources easily."

---

copy data from on-premise (access via data gateway) SQL Server database to Warehouse -> use data pipeline

---

Workspace1 contains a warehouse named Warehouse1.
deploy Warefouse1 to a new Workspace2.
As part of the deployment process, you need to verify whether Warehouse1 contains invalid references. The solution must minimize development effort.
-> use data pipeline

---

\_`Managed private endpoint`: This is a networking feature, not an authentication feature.
It secures the "tunnel" through which data travels, but it doesn't verify the identity of the caller.

\_`An on-premises data gateway` = `data management gateway`: is used for data sources that are physically located on-premises or on a private VM.
While it works for Dataflows and Pipelines, it is not the primary way for a Spark Job to connect to an Azure-native service like Azure SQL

---

The roles in Fabric:

1. **Fabric Admin** (The "CEO"):
   Has access to the Admin Portal.
   Can create Top-Level Domains.
   Can delegate power to others.
   Use this when: The task involves tenant settings, creating domains, or capacity management.
2. **Domain Admin** (The "Department Manager"):
   Appointed by the Fabric Admin to manage a specific domain (e.g., Sales).
   Can create Subdomains (e.g., East, West).
   Can edit domain settings and icons.
   Can assign other people as Domain Admins or Contributors.
   Use this when: The domain already exists and you need to organize things inside it.
3. **Domain Contributor** (The "Organizer"):
   Can assign workspaces to the domain.
   They cannot create subdomains or change domain-level settings.
   Use this when: You just need someone to link their workspaces to a specific domain.
4. **Workspace Admin** (The "Team Leader"):
   Has full control over the content inside a specific workspace (Notebooks, Pipelines, Reports).
   Has no power over Domains or Subdomains unless they are also given a Domain role.

---

`Member` = Contributor + share items + add members + update workspace app

---

`endorsement`:

- **Promoted** = "The Artist's Choice." The person who made it says, "Hey, check this out, it's pretty good!"
- **Certified** = "The Health Inspector's Grade." An outside authority came in, checked the kitchen, and gave it an 'A' rating

---

**Domain roles vs Workspace roles**

IMPORTANT: Domain permissions DO NOT grant Workspace permissions
\_When added to a domain, it is by default CONTRIBUTOR

---

In Microsoft Fabric, the `Monitor hub` (accessible from the left-hand navigation rail) is the centralized location
for tracking the operational health and execution details of all items, including Spark Notebooks.

- `Real-time hub`: This is used for managing data in motion (Eventstreams, KQL tables).
- `OneLake data hub`: This is a "discovery" tool. It helps you find and browse items (Lakehouses, Warehouses) across the tenant.
- The `Admin monitoring workspace`: This workspace provides high-level reports on tenant usage, adoption, and auditing (who created what).
- `Microsoft Fabric Capacity Metrics app`: This app is strictly for cost and performance monitoring.
  It shows you how many "Capacity Units" (CUs) your notebook consumed and if you are being throttled.

---

- `Semantic Link` (specifically the `sempy` library in Python) is the bridge between Data Engineering (Notebooks) and Power BI (Semantic Models).
  - you can trigger a refresh of a semantic model directly from a Spark Notebook
  - programmatically poll the status, see the current status (In Progress, Completed, Failed) and even specific details about the refresh duration and type.
  - include a Power BI refresh at the end of an ETL pipeline - once your Spark job finishes loading the Gold tables, the notebook can "dynamically" tell the semantic model to update.

---

- convert(varchar, purchase_date, 7) -> in format mon dd yyyy

- convert(varchar, purchase_date, 12) -> in format yyyymmdd

---

In Microsoft Fabric, a `semantic model` acts as the business-friendly layer, translating raw data from sources like Lakehouses/Warehouses
into understandable tables, relationships, measures, and business logic for reporting and analysis,
essentially bridging the gap between technical data and user insights, powering Power BI reports, and enabling advanced data science via Semantic Link.

---

Fabric items that can be endorsed:

1. Data Storage Items
   These are the most commonly endorsed items because they serve as the "Single Source of Truth."
   Lakehouse
   Data Warehouse
   KQL Database (within an Eventhouse)
   Mirrored Database
2. Data Transformation & Engineering Items
   You endorse these to show that the code or logic used to build the data is production-ready.
   Dataflow Gen2
   Notebook
   Data Pipeline
   Spark Job Definition
3. Analytics & Reporting Items
   These are the items end-users interact with most frequently.
   Semantic Model (Crucial for Direct Lake scenarios)
   Report (Power BI)
   Dashboard
   Datamart
   Paginated Report
4. Real-Time Intelligence Items
   Eventstream
   Real-Time Dashboard
5. Data Science Items
   Machine Learning Model
   Machine Learning Experiment

---

`V-Order` is a Fabric optimization that physically reorganizes data. However, V-Order adds overhead during writes.
In a Data Warehouse, it is enabled by default.

---

When sharing a lakehouse in Microsoft Fabric, the default is minimal access, granting
ReadData (SQL endpoint access), ReadAll (Spark access to all files/tables), or Build (Power BI report creation on the default dataset).

---

`Deployment pipelines` in Fabric and Power BI move metadata (definitions), not the actual data stored within the items.
When you deploy a semantic model, you are moving the schema, measures, and relationships. The data remains in the source environment.
