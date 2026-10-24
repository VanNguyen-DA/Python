Sales & Distribution Analytics Platform
Overview

This project builds an end-to-end Analytics Engineering pipeline using the Medallion Architecture (Bronze → Silver → Gold). The solution ingests raw distributor and sales data into PostgreSQL, transforms and validates data using SQL, and creates analytical data marts for business reporting.

Tech Stack
Python: Data ingestion, profiling, and ETL orchestration
PostgreSQL: Data warehouse and SQL transformations
VS Code: Development environment
Table of Contents
Background Overview
Dataset Description & Data Structure
Analysis Process
Final Conclusion & Recommendations
1. Background Overview
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
2. Dataset Description & Data Structure
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
Column	Descriptioncustomer_id	Unique customer identifier
customer_name	Customer name
customer_type	Customer classification
channel	Sales channel
province	Customer province
region	Business region
join_date	Customer registration date
credit_limit	Approved credit limit
status	Customer status
3. Analysis Process

The project follows a Medallion Architecture approach.

Step 1. Data Profiling (Python)

Raw data is explored using Python before any transformation.

Objectives
Understand data structure
Identify missing values
Detect duplicated records
Examine data quality issues

Example:

import pandas as pd

customers = pd.read_csv("customer_master.csv")

print(customers.info())
print(customers.isna().sum())
print(customers.duplicated().sum())

Step 2. Bronze Layer (Raw Ingestion)
Tool
Python
PostgreSQL
SQLAlchemy
Objectives

Load source files into PostgreSQL without modification.

All columns are initially loaded as text and enriched with metadata:

Metadata Column	Description_source_file	Original file name
_source_platform	Data source location
_ingested_at	Ingestion timestamp
_batch_id	Pipeline execution ID

Example:

customers.to_sql(
    name='customer_master',
    con=engine,
    schema='raw',
    if_exists='replace',
    index=False
)

Step 3. Silver Layer (Data Cleaning & Standardization)
Tool
PostgreSQL
SQL
Objectives

Transform raw data into clean and reliable datasets.

Customer Master Example
Data Type Standardization
join_date::DATE
credit_limit::NUMERIC(18,2)

Remove Invalid Records
WHERE customer_id IS NOT NULL
AND TRIM(customer_id) <> ''

Deduplicate Customers
ROW_NUMBER() OVER (
PARTITION BY customer_id
ORDER BY _ingested_at DESC
)

Create Staging Table
CREATE TABLE staging.stg_customer_master AS
SELECT *
FROM deduped
WHERE rn = 1;

Data Quality Validation
Primary Key Validation
ALTER TABLE staging.stg_customer_master
ADD PRIMARY KEY (customer_id);

Duplicate Check
SELECT customer_id,
COUNT(*)
FROM staging.stg_customer_master
GROUP BY customer_id
HAVING COUNT(*) > 1;

Step 4. Gold Layer (Business Data Mart)

Business-ready marts are created to support reporting and analytics.

Example: Distributor Performance Mart

Metrics include:

Total Orders Placed
Total Quantity Ordered
Total Quantity Delivered
Fill Rate %
Delivered Revenue
On-Time Delivery %
Distributor Performance KPIs

Example:

CREATE TABLE dwh.mart_distributor_performance AS
SELECT
    distributor_id,
    order_month,
    COUNT(DISTINCT order_id) AS total_orders_placed,
    SUM(qty_ordered) AS total_qty_ordered,
    SUM(qty_delivered) AS total_qty_delivered
FROM staging.stg_distributor_orders
GROUP BY distributor_id, order_month;

4. Final Conclusion & Recommendations
Key Deliverables

✅ Built an end-to-end Analytics Engineering pipeline

✅ Implemented Medallion Architecture:

Bronze (Raw)
Silver (Cleaned)
Gold (Business Ready)

✅ Automated data ingestion with Python

✅ Applied data quality checks and validation using SQL

✅ Developed business-focused data marts for reporting and dashboarding

Recommendations
Implement incremental loading for production-scale datasets
Add automated data quality monitoring
Schedule ETL jobs using Apache Airflow
Integrate CI/CD pipeline for deployment
Expand Power BI dashboards with advanced KPI tracking
Project Architecture
CSV Sources
     │
     ▼
Python Ingestion
     │
     ▼
PostgreSQL (Bronze)
     │
     ▼
SQL Transformation
     │
     ▼
PostgreSQL (Silver)
     │
     ▼
Business Data Marts (Gold)
     │
     ▼
Power BI Dashboard


Tools Used: Python • PostgreSQL • SQLAlchemy • VS Code • SQL • Power BI

