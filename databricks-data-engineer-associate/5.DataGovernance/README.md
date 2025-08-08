# 5. Data Governance
## Databricks SQL 
When you enable the **Serverless** option in `SQL Warehouse`, you’re telling Databricks:

> I don’t want to manage clusters — just give me a fast, elastic compute environment and only charge me for what I use.
<br> -> instant startup | ad hoc sql queries | auto scaling | dashboard queries

![alt text](./img/image-8.png)

### SQL Editor 
In `SQL Editor`, you can write a query and save that query. You can also create dashboards based on that query. 

Additionally, you can schedule the query: `Schedule` -> `Add schedule`.

### Alerts
In `Alerts`, to create an alert, you MUST have a query. <br>
The way `Alerts` work is that, they take the output from a query, and based on the output, you can **define a condition** and **trigger an alert** if the condition evaluates to true. 

E.g., there is an alert if the **total_amount** exceeds 50k. <br>
We first `Create alert` -> name the alert and choose the corresponding query -> adjust the `Trigger condition`


## Intro to Data Governance
Databricks delivers **data governance** using a set of integrated tools that ensure security, access control, compliance, **data quality and auditability**. 
> Unity Catalog is the core governance product in Databricks. It is a Databricks offered **unified solution** for implementing data governance in the Data Lakehouse.

## Data Governance with Unity Catalog
![alt text](./img/image.png)
- `Data Access Control`: restrict access to only required users and groups.
- `Data Audit`: how the user is using the data and how often and when it's being accessed -> available audit logs (delivered as JSON files to a configurable **cloud storage location** *(that you own and not stored inside Databricks)*).
- `Data Lineage`: trace back the data to its source to ensure authenticity; provides lineage both downstream and upstream in a pipeline
- `Data Discoverability`: Databricks makes a **searchable data catalog** with the help of Unity Catalog

An account **level Unity Catalog** that contains user management and the metadata management, where `User Management` and `(Unity Catalog) Metastore` are centralized. 
![alt text](./img/image-1.png)

### Demo 
Access `Catalog` on the left tab to modify -> use the searching bar in the UI to look for the desired tables/schemas.

![alt text](./img/image-2.png)

To see `Data lineage`, click `See lineage graph` and you will see the complete DAG to the table. 

![alt text](./img/image-3.png)

![alt text](./img/image-4.png)

## Security model
![alt text](./img/image-5.png)

- `Account admin` is the highest level user in Unity Catalog as they have full access to **EVERY metastore** associated to the Databricks account.
- `Metastore admin` will have similar capabilities to Account Admin, but they can only access the metastore that they're administrator for.
- Each `object` within metastore has an `owner` (by default, the user/service principal that creates the object)
- `Object user`: users in the Enterprise

### Access Control List 
_To grant a privilege:
```sql
GRANT <privilege> 
ON <securable_object>
TO <principal>

GRANT SELECT 
ON TABLE customers
TO sales
```

_To remove a privilege:
```sql
REVOKE <privilege>
ON <securable_object>
FROM <identity>

REVOKE SELECT 
ON TABLE customers 
FROM sales
```
In case we want to use the `UI`, click on the `Metastore object` (e.g., table) that we want to modify the privilege -> choose `permission` tab -> type in the **principals** (i.e., the users) and tick on the desired privileges.


### Hierarchical Access Control
_We need to first provide `USE CATALOG` and `USE SCHEMA` for the users in order to `SELECT table`.

![alt text](./img/image-6.png)

---

**Privilege inheritance**: it makes the privileges for the *child objects* available at the *parent level* in the hierarchy.

So you can grant SELECT privilege on all the tables **in a schema** with one statement. <br>
-> this will also be applicable for the current and future tables created in that **schema**.

![alt text](./img/image-7.png)

For more info about different privilege types, refer to [Privilege types by securable object in Unity Catalog](https://docs.databricks.com/aws/en/data-governance/unity-catalog/manage-privileges/privileges#privilege-types-by-securable-object-in-unity-catalog).

## Delta Sharing with Unity Catalog
Delta Sharing allows you to share your data across the organization (even to external organization). 
> Delta Sharing is READ-ONLY permission.

There are 2 modes:
- `Open sharing`: share to ones who dont use Databricks (e.g., PowerBI, Apache Spark...). There is also a library for this.
- `Databircks-to-Databricks sharing`

### Enable Delta Sharing on Metastore
*Only required if you are sharing data outside (external) your organization*.

Access `Catalog` tab on main page -> choose your metastore -> tick `Allow Delta Sharing with parties outside your organization`.

### Delta Sharing permission
In order to work with **Delta Sharing**, you need these permissions (mandatory) to share the data: `CREATE PROVIDER`, `CREATE RECIPIENT`, `CREATE SHARE`. 
<br> If you want to **use the shared data**, you need `USE PROVIDER`, `USE RECIPIENT`, `USE SHARE`.

### How to share data (from provider side)?
Access `Catalog` tab on main page -> choose `Delta Sharing` -> click on `New recepient` button -> type in the recipient's name and **sharing identifer** *(Databricks only)* -> `Create`. 
> So now, a new recipient has been created.

![alt text](image.png)

Go back to `Delta Sharing` -> go to `Shared by me` tab -> click on `Share data` button -> fill in all required information including shared name, data assets, notebooks, etc ...
![alt text](image-1.png)

### How to use (from recipient side)?
First, grant all 3 permissions required for the recipient (using Unity Catalog). 

Then, go to `Delta Sharing` -> choose the corresponding provider in the list -> `Create the catalog` (so that the recipient can get the shared data via this **catalog**).

---
### Monitor and manage Delta Sharing egress costs (for providers)
> Delta Sharing does not require data replication. 
It means that your cloud vendor may charge data egress fees when you share data **across clouds or regions**.

In Databricks Marketplace, you can clone and use 2 notebooks to monitor egress usage patterns and costs associated with Delta Sharing:
- `IP Ranges Mapping Pipeline` notebook
- `Egress Cost Analysis Pipeline` notebook

## Lakehouse Federation
- Lakehouse Federation is the query federation platform that enables you to use Databricks to **run queries** against **multiple external data sources** (withou actually migrating data to Lakehouse).
- Lakehouse Federation allows only to read/query your data; NO manipulation (update, insert, delete, etc) is allowed. 

**Use Case**: You have a critical operational database (e.g., *PostgreSQL* for user profiles) that you cannot migrate. **Lakehouse Federation** lets you run queries that join a table in **Databricks** with a table in PostgreSQL in a single statement, without needing to build a complex ETL pipeline to first copy the PostgreSQL data into Databricks.

### Create External Connections in Databricks / Create Lakehouse Federation
Access `Catalog` tab on main page -> choose `External Data` -> `Create connection` button -> Fill out the information for connection basics, authentication and catalog basics. 