# 5. Data Governance
## Databricks SQL 
When you enable the **Serverless** option in `SQL Warehouse`, you’re telling Databricks:

> I don’t want to manage clusters — just give me a fast, elastic compute environment and only charge me for what I use.
<br> -> instant startup | ad hoc sql queries | auto scaling | dashboard queries

### SQL Editor 
In `SQL Editor`, you can write a query and save that query. Additionally, you can create dashboards based on that query. 

### Alerts
In `Alerts`, to create an alert, you MUST have a query. <br>
The way `Alerts` work is that, they take the output from a query, and based on the output, you can **define a condition** and **trigger an alert** if the condition evaluates to true. 

E.g., there is an alert if the **total_amount** exceeds 50k. <br>
We first `Create alert` -> name the alert and choose the corresponding query -> adjust the `Trigger condition`


