# 🚚 Sales & Distribution Analytics Platform

## Overview

This project builds an end-to-end Analytics Engineering pipeline using the **Medallion Architecture (Bronze → Silver → Gold)**.

The solution ingests raw distributor and sales data into PostgreSQL, transforms and validates data using SQL, and creates analytical data marts for business reporting.

---

## 🛠️ Tech Stack

- **Python**: Data ingestion, profiling, and ETL orchestration
- **PostgreSQL**: Data warehouse and SQL transformations
- **VS Code**: Development environment

---

## 📋 Table of Contents

1. [Background Overview](#-background-overview)
2. [Dataset Description & Data Structure](#-dataset-description--data-structure)
3. [Analysis Process](#-analysis-process)
4. [Final Conclusion & Recommendations](#-final-conclusion--recommendations)

---

# 📖 Background Overview

## What is this project about?

The project aims to build a modern data warehouse for sales and distribution analytics. Raw operational data is collected from multiple source systems, loaded into PostgreSQL, cleaned and standardized through a Medallion Architecture, and transformed into business-ready datasets.

## Business Questions

The project helps answer key business questions such as:

- Which distributors have the highest delivery performance?
- What is the actual fill rate by distributor?
- Which regions generate the highest sales volume?
- How effectively are orders delivered on time?
- How does sales performance compare against business targets?

## Who is this project for?

- Business Analysts
- Data Analysts
- Analytics Engineers
- Supply Chain Teams
- Sales Managers
- Distribution Managers

---

# 🗂️ Dataset Description & Data Structure

## Data Source

The project uses multiple CSV source files representing:

- Customer Master
- Product Master
- Distributor Master
- Employee Master
- Sales Transactions
- Orders
- Sales Targets

## Data Warehouse Architecture

```text
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
```

## Example Data Structure

### Customer Master

| Column | Description |
|----------|-------------|
| customer_id | Unique customer identifier |
| customer_name | Customer name |
| customer_type | Customer classification |
| channel | Sales channel |
| province | Customer province |
| region | Business region |
| join_date | Customer registration date |
| credit_limit | Approved credit limit |
| status | Customer status |

---

# ⚙️ Analysis Process

The project follows a Medallion Architecture approach.

## Step 1: Data Profiling (Python)

Raw data is explored using Python before any transformation.

### Objectives

- Understand data structure
- Identify missing values
- Detect duplicated records
- Examine data quality issues

### Example

```python
import pandas as pd

customers = pd.read_csv("customer_master.csv")

print(customers.info())
print(customers.isna().sum())
print(customers.duplicated().sum())
```

---

## Step 2: Bronze Layer (Raw Ingestion)

### Tools

- Python
- PostgreSQL
- SQLAlchemy

### Objective

Load source files into PostgreSQL without modification.

All columns are initially loaded as text and enriched with metadata.

### Metadata Columns

| Column | Description |
|----------|-------------|
| _source_file | Original file name |
| _source_platform | Data source location |
| _ingested_at | Ingestion timestamp |
| _batch_id | Pipeline execution ID |

### Example

```python
customers.to_sql(
    name='customer_master',
    con=engine,
    schema='raw',
    if_exists='replace',
    index=False
)
```

---

## Step 3: Silver Layer (Data Cleaning & Standardization)

### Tools

- PostgreSQL
- SQL

### Objective

Transform raw data into clean and reliable datasets.

### Customer Master Example

#### 1. Data Type Standardization

```sql
join_date::DATE
credit_limit::NUMERIC(18,2)
```

#### 2. Remove Invalid Records

```sql
WHERE customer_id IS NOT NULL
AND TRIM(customer_id) <> ''
```

#### 3. Deduplicate Customers

```sql
ROW_NUMBER() OVER (
    PARTITION BY customer_id
    ORDER 
