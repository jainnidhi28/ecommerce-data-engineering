# Azure E-Commerce Data Engineering Platform

## Project Overview

An end-to-end Azure Data Engineering platform built to ingest, process, validate, transform, and serve e-commerce data using modern cloud data engineering technologies.

The platform follows a Medallion Architecture with Bronze, Silver, and Gold layers and includes incremental processing, CDC, SCD Type 2, data quality validation, data quarantine, audit logging, monitoring, Delta Lake, performance optimization, and Git-based version control.

## Architecture

DATA SOURCES
    |
    v
Azure Data Factory
    |
    v
ADLS Gen2 - Bronze
    |
    v
Azure Databricks + PySpark
    |
    +--> Data Quality --> Quarantine
    |
    v
Silver - Delta Lake
    |
    v
Gold - Delta Lake
    |
    v
Synapse Serverless SQL Views

## Technologies Used

- Azure Data Lake Storage Gen2
- Azure Data Factory
- Azure Databricks
- PySpark
- Delta Lake
- Azure Synapse Analytics - Serverless SQL
- GitHub
- Git / Version Control

## Data Sources

Five e-commerce datasets are used.

### Customers
customer_id, name, email, city, country, signup_date

### Products
product_id, product_name, category, price

### Orders
order_id, customer_id, order_date, product_id, quantity, amount, status

### Payments
payment_id, order_id, payment_date, payment_method, amount, status

### Website Events
event_id, customer_id, event_type, timestamp, product_id

## Data Pipeline

Azure Data Factory is used as the ingestion and orchestration layer.

The pipeline supports:
- Metadata-driven ingestion
- Dynamic file processing
- Incremental ingestion
- Pipeline monitoring
- Failure tracking

The metadata-driven pipeline dynamically processes multiple source files using a ForEach activity instead of creating separate pipelines for every dataset.

## Bronze Layer

The Bronze layer stores ingested source data in ADLS Gen2.

bronze/
- customers.csv
- products.csv
- orders.csv
- payments.csv
- website_events.csv

The Bronze layer preserves incoming data before business transformations.

## Silver Layer

Azure Databricks and PySpark are used to clean and standardize Bronze data.

Transformations include:
- Duplicate removal
- Null handling
- String trimming
- Email standardization
- Data type conversion
- Date conversion
- Timestamp conversion
- Status normalization
- Data validation

Cleaned data is stored as Delta tables:

silver/
- customers
- products
- orders
- payments
- website_events

## Gold Layer

The Gold layer contains business-ready datasets.

### Sales Daily
- Total orders
- Total quantity
- Total revenue
- Average order value

### Product Performance
- Product-level orders
- Quantity sold
- Revenue
- Average order value

### Revenue by City
- Orders by city
- Revenue by city
- Average order value

### Customer 360
Combines customer information with:
- Order metrics
- Total spend
- Average order value
- Last order date
- Website activity
- Unique event types

### Order Metrics
Combines order, customer, and product information.

## Incremental Loading

Incremental processing was implemented to avoid processing the complete dataset every time new data arrives.

source/incremental
    |
    v
Azure Data Factory
    |
    v
bronze/incremental
    |
    v
Databricks
    |
    v
Delta MERGE
    |
    v
Silver Orders

Delta Lake MERGE is used to insert new records and update existing records based on the business key.

## Change Data Capture (CDC)

CDC processing was demonstrated using Delta MERGE.

Example:

Existing Order:
O0502 -> status = completed

CDC Update:
O0502 -> status = cancelled
O0522 -> new order

The MERGE operation updates matching records and inserts new records.

## Slowly Changing Dimension Type 2

SCD Type 2 was implemented for customer history tracking.

When a customer's attribute changes, the historical version is preserved instead of simply overwriting it.

Example:

Customer C0008

Old Version:
city = Pune
is_current = false

New Version:
city = Mumbai
is_current = true

The implementation maintains:
- effective_from
- effective_to
- is_current

## Data Quality

A dedicated data-quality framework validates Silver orders.

Checks include:
- Null order IDs
- Duplicate order IDs
- Invalid amounts
- Invalid quantities
- Invalid statuses
- Referential integrity

Invalid records are identified and separated from valid records.

## Data Quarantine

Invalid records are quarantined instead of silently discarded.

Rejected records contain:
- rejection_reason

Example reasons:
- Invalid amount
- Invalid quantity

Quarantine location:
silver/quarantine/orders

## Audit Logging

Pipeline execution information is stored in a Delta audit table.

The audit log captures:
- pipeline_name
- table_name
- run_time
- rows_processed
- rows_rejected
- status
- error_message

## Performance Optimization

Spark optimization techniques demonstrated include:

### Broadcast Join

Small dimension tables can be broadcast to reduce shuffle during joins.

### Partitioning

Gold data was partitioned by order_date to support partition pruning.

### Query Plan Analysis

Spark execution plans were inspected using explain(True).

Other considerations:
- Shuffle reduction
- Appropriate partitioning
- Avoiding unnecessary broadcasts
- Avoiding over-partitioning

## Synapse Serverless

Azure Synapse Serverless SQL is used as a SQL access layer over Gold Delta data.

Gold Delta datasets are queried using OPENROWSET with FORMAT = 'DELTA'.

SQL views:
- vw_sales_daily
- vw_product_performance
- vw_revenue_by_city
- vw_customer_360
- vw_order_metrics

This provides a SQL interface without provisioning a Dedicated SQL Pool.

## Monitoring

### Azure Data Factory
Used to monitor:
- Pipeline runs
- Activity status
- Duration
- Errors
- Copy activity execution

### Databricks
Used to monitor:
- Notebook execution
- Job runs
- Logs
- Failures

### Audit Layer
Delta audit logs provide additional pipeline-level execution history.

## Git and Version Control

GitHub is used for version control.

ADF is connected to GitHub and stores ADF resources as JSON definitions.

Databricks notebooks are maintained inside the same GitHub repository.

ADF uses the adf_publish branch for generated deployment artifacts.

## Repository Structure

ecommerce-data-engineering/
|
+-- databricks/
|   +-- bronze_to_silver/
|   +-- silver_to_gold/
|   +-- incremental/
|   +-- scd2/
|   +-- data_quality/
|   +-- performance/
|
+-- dataset/
+-- factory/
+-- linkedService/
+-- pipeline/
+-- publish_config.json

## Key Engineering Concepts Demonstrated

- Cloud Data Engineering
- Azure Data Lake
- ETL / ELT
- Data Orchestration
- Metadata-driven pipelines
- Batch Processing
- Incremental Processing
- Change Data Capture
- Medallion Architecture
- Delta Lake
- PySpark
- Data Cleaning
- Data Quality
- Data Quarantine
- SCD Type 2
- Data Modeling
- Audit Logging
- Monitoring
- Spark Optimization
- Serverless SQL
- Git / GitHub
- Version Control

## Future Improvements

Potential production enhancements:
- Automated CI/CD deployment
- Automated data-quality alerting
- Schema evolution handling
- Automated testing
- Azure Key Vault integration
- Parameterized environments
- Development / Test / Production environments
- Infrastructure as Code
- Automated pipeline scheduling

## Project Objective

The objective is to demonstrate how a modern Azure-based data platform can ingest raw e-commerce data, transform it using distributed processing, maintain historical and incremental changes, enforce data quality, and expose curated datasets through a SQL analytics layer.
