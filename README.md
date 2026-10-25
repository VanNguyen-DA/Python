# Sales & Distribution Analytics Platform

## Overview

This project demonstrates an end-to-end Analytics Engineering solution using the **Medallion Architecture (Bronze → Silver → Gold)**.

The pipeline ingests raw Sales & Distribution data from Excel/CSV files, loads it into PostgreSQL, applies data quality transformations, and builds business-ready data marts for Power BI reporting.

---

## Tech Stack

- Python 3.11
- Pandas
- PostgreSQL 16
- SQLAlchemy
- Power BI Desktop
- Git & GitHub
- VS Code

---

## Project Architecture

```text
Source Files
 ├── SRC01_sales_transactions
 ├── SRC02_sales_target_plan
 ├── SRC03_customer_master
 ├── SRC04_product_master
 ├── SRC05_distributor_orders
 ├── SRC06_distributor_master
 ├── SRC07_employee_master
 ├── SRC08_territory_mapping
 ├── SRC09_return_transactions
 └── SRC10_promotion_program

            │
            ▼

      BRONZE LAYER
      (raw schema)

            │
            ▼

      SILVER LAYER
     (staging schema)

            │
            ▼

       GOLD LAYER
       (dwh schema)

            │
            ▼

      Power BI Dashboard
```

---

# Data Sources

| Source | Description |
|----------|----------|
| SRC01 | Sales Transactions |
| SRC02 | Sales Target Plan |
| SRC03 | Customer Master |
| SRC04 | Product Master |
| SRC05 | Distributor Orders |
| SRC06 | Distributor Master |
| SRC07 | Employee Master |
| SRC08 | Territory Mapping |
| SRC09 | Return Transactions |
| SRC10 | Promotion Program |

---

# ETL Workflow

## Phase 1 — Bronze Layer (Raw Data Ingestion)

### Objective

Load source data exactly as received without modification.

### Step 1 — Read CSV File

Example: Customer Master

```python
customers = pd.read_csv(
    "/Raw/SRC03_customer_master.csv"
)
```

### Step 2 — Convert All Columns to Text

Bronze layer stores raw values to avoid ingestion failures.

```python
customers = customers.astype(str)
```

### Step 3 — Add Metadata Columns

```python
customers['_source_file'] = 'SRC03_customer_master.csv'
customers['_source_platform'] = 'local_storage'
customers['_ingested_at'] = datetime.now()
customers['_batch_id'] = str(uuid.uuid4())
```

### Step 4 — Load Into PostgreSQL

```python
customers.to_sql(
    name='customer_master',
    con=engine,
    schema='raw',
    if_exists='replace',
    index=False
)
```

### Result

```text
raw.customer_master
```

Example Bronze Data:

| customer_id | customer_name | status | _source_file |
|------------|------------|------------|------------|
| C001 | ABC Store | Active | SRC03_customer_master.csv |
| C002 | XYZ Shop | Active | SRC03_customer_master.csv |

---

### Bronze Load Script

```python
import os
import uuid
import pandas as pd

from datetime import datetime
from dotenv import load_dotenv
from sqlalchemy import create_engine

load_dotenv()

connection_string = (
    f"postgresql+psycopg2://"
    f"{os.getenv('DB_USER')}:"
    f"{os.getenv('DB_PASSWORD')}@"
    f"{os.getenv('DB_HOST')}:"
    f"{os.getenv('DB_PORT')}/"
    f"{os.getenv('DB_NAME')}"
)

engine = create_engine(connection_string)

customers = pd.read_csv(
    "SRC03_customer_master.csv"
)

customers = customers.astype(str)

customers['_source_file'] = "SRC03_customer_master.csv"
customers['_source_platform'] = "local_storage"
customers['_ingested_at'] = datetime.now()
customers['_batch_id'] = str(uuid.uuid4())

customers.to_sql(
    "customer_master",
    schema="raw",
    con=engine,
    if_exists="replace",
    index=False
)
```

---

## Phase 2 — Silver Layer (Data Cleaning & Standardization)

### Objective

Transform raw tables into trusted datasets.

### Step 1 — Trim Spaces

```sql
TRIM(customer_name)
```

### Step 2 — Convert Data Types

```sql
join_date::DATE

credit_limit::NUMERIC(18,2)
```

### Step 3 — Remove Invalid Records

```sql
WHERE customer_id IS NOT NULL
```

### Step 4 — Remove Duplicates

Keep latest record based on ingestion timestamp.

```sql
ROW_NUMBER() OVER (
PARTITION BY customer_id
ORDER BY _ingested_at DESC
)
```

### Step 5 — Create Primary Key

```sql
ALTER TABLE staging.stg_customer_master
ADD PRIMARY KEY(customer_id);
```

---

### Silver SQL Model

```sql
DROP TABLE IF EXISTS staging.stg_customer_master;

CREATE TABLE staging.stg_customer_master AS

WITH deduped AS (

    SELECT
        TRIM(customer_id)::VARCHAR(50) AS customer_id,
        TRIM(customer_name)::VARCHAR(255) AS customer_name,
        TRIM(customer_type)::VARCHAR(100) AS customer_type,
        TRIM(channel)::VARCHAR(100) AS channel,
        TRIM(province)::VARCHAR(100) AS province,
        TRIM(region)::VARCHAR(100) AS region,
        TRIM(address)::VARCHAR(500) AS address,
        TRIM(phone)::VARCHAR(50) AS phone,
        TRIM(tax_code)::VARCHAR(50) AS tax_code,
        join_date::DATE AS join_date,
        credit_limit::NUMERIC(18,2) AS credit_limit,
        TRIM(status)::VARCHAR(50) AS status,

        _source_file,
        _source_platform,
        _ingested_at,
        _batch_id,

        ROW_NUMBER() OVER (
            PARTITION BY customer_id
            ORDER BY _ingested_at DESC
        ) AS rn

    FROM raw.customer_master

    WHERE customer_id IS NOT NULL
)

SELECT *

FROM deduped

WHERE rn = 1;
```

### Result

```text
staging.stg_customer_master
```

Trusted customer dimension source.

---

## Phase 3 — Gold Layer (Business Analytics)

### Objective

Create business-ready data marts and KPI tables.

### Step 1 — Build Fact Tables

```text
fact_sales
fact_returns
fact_targets
```

### Step 2 — Build Dimension Tables

```text
dim_customers
dim_products
dim_employees
dim_distributors
dim_date
```

### Step 3 — Build Data Mart

```text
mart_sales_vs_target
mart_distributor_performance
```

---

## Example Gold Model

Distributor Performance Mart

```sql
DROP TABLE IF EXISTS dwh.mart_distributor_performance;

CREATE TABLE dwh.mart_distributor_performance AS

SELECT

    distributor_id,
    order_month,
    order_quarter,

    COUNT(DISTINCT order_id) AS total_orders_placed,

    SUM(qty_ordered) AS total_qty_ordered,

    SUM(qty_delivered) AS total_qty_delivered,

    CASE
        WHEN SUM(qty_ordered) > 0
        THEN ROUND(
            (
                SUM(qty_delivered)::NUMERIC
                /
                SUM(qty_ordered)
            ) * 100,
            2
        )
        ELSE 0
    END AS actual_fill_rate_pct,

    SUM(gross_amount) AS total_gross_value,

    SUM(delivered_amount) AS total_delivered_value,

    COUNT(
        DISTINCT CASE
            WHEN ontime_delivery = 'Yes'
            THEN order_id
        END
    ) AS ontime_orders_count,

    CASE
        WHEN COUNT(DISTINCT order_id) > 0
        THEN ROUND(
            (
                COUNT(
                    DISTINCT CASE
                    WHEN ontime_delivery = 'Yes'
                    THEN order_id
                    END
                )::NUMERIC
                /
                COUNT(DISTINCT order_id)
            ) * 100,
            2
        )
        ELSE 0
    END AS ontime_delivery_rate_pct

FROM staging.stg_distributor_orders

GROUP BY
    distributor_id,
    order_month,
    order_quarter;
```

---

# Data Warehouse Model

## Bronze Schema

```text
raw.customer_master
raw.product_master
raw.sales_transactions
raw.distributor_orders
raw.employee_master
raw.distributor_master
raw.territory_mapping
raw.return_transactions
raw.sales_target_plan
raw.promotion_program
```

## Silver Schema

```text
staging.stg_customer_master
staging.stg_product_master
staging.stg_sales_transactions
staging.stg_distributor_orders
staging.stg_employee_master
staging.stg_distributor_master
staging.stg_territory_mapping
staging.stg_return_transactions
staging.stg_sales_targets
staging.stg_promotion_program
```

## Gold Schema

```text
dwh.dim_customers
dwh.dim_products
dwh.dim_employees
dwh.dim_distributors
dwh.dim_date

dwh.fact_sales
dwh.fact_returns
dwh.fact_targets

dwh.mart_sales_vs_target
dwh.mart_distributor_performance
```

---

# Power BI Dashboard

Main reporting modules:

### Sales Performance

- Revenue Trend
- Sales by Region
- Sales by Product
- Sales by Customer

### Target Achievement

- Actual vs Target
- Achievement %
- Variance Analysis

### Distributor Performance

- Fill Rate %
- On-Time Delivery %
- Order Fulfillment Rate

### Return Analysis

- Return Quantity
- Return Amount
- Return Rate

---

# Key Skills Demonstrated

- ETL Pipeline Development
- PostgreSQL Data Warehousing
- Medallion Architecture
- Data Quality Validation
- SQL Data Modeling
- Star Schema Design
- Data Mart Development
- Power BI Reporting
- Analytics Engineering
