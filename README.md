Sales & Distribution Analytics Platform
Overview
This project builds an end-to-end Analytics Engineering pipeline using the Medallion Architecture (Bronze → Silver → Gold).

The solution ingests raw distributor and sales data into PostgreSQL, transforms and validates data using SQL, and creates analytical data marts for business reporting.

🛠️ Tech Stack
Python: Data ingestion, profiling, and ETL orchestration
PostgreSQL: Data warehouse and SQL transformations
VS Code: Development environment
📋 Table of Contents
Background Overview
Dataset Description & Data Structure
Analysis Process
Final Conclusion & Recommendations
📖 Background Overview
What is this project about?
The project aims to build a modern data warehouse for sales and distribution analytics. Raw operational data is collected from multiple source systems, loaded into PostgreSQL, cleaned and standardized through a Medallion Architecture, and transformed into business-ready datasets.

Business Questions
The project helps answer key business questions such as:

Which distributors have the highest delivery performance?
What is the actual fill rate by distributor?
Which regions generate the highest sales volume?
How effectively are orders delivered on time?
How does sales performance compare against business targets?
Who is this project for?
Business Analysts
Data Analysts
Analytics Engineers
Supply Chain Teams
Sales Managers
Distribution Managers
🗂️ Dataset Description & Data Structure
Data Source
The project uses multiple CSV source files representing:

Customer Master
Product Master
Distributor Master
Employee Master
Sales Transactions
Orders
Sales Targets
Data Warehouse Architecture
Raw Files (CSV)
      │
      ▼
Bronze Layer (raw schema)
      │
      ▼
Silver Layer (staging schema)
      │
      ▼
Gold Layer (dwh schema)
      │
      ▼
Power BI Dashboard
Example Data Structure
Customer Master
Column	Description
customer_id	Unique customer identifier
customer_name	Customer name
customer_type	Customer classification
channel	Sales channel
province	Customer province
region	Business region
join_date	Customer registration date
credit_limit	Approved credit limit
status	Customer status
⚙️ Analysis Process
