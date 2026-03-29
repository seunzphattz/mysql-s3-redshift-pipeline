# End-to-End Data Pipeline: MySQL → S3 → Redshift (AWS)

## Overview

## Overview

This project demonstrates a production-style ELT data pipeline that extracts operational data from MySQL (Amazon Aurora), stages it in Amazon S3, and transforms it into analytics-ready datasets in Amazon Redshift.
The solution addresses common challenges such as overloading operational databases, slow query performance, and delayed dashboards by separating transactional workloads from analytical processing.

By decoupling operational systems from analytics, the pipeline ensures improved performance, scalability, and timely insights, while supporting incremental data loads and high-volume processing.

---

## Problem Statement

Overloaded operational DB
Slow queries
Poor dashboards
Need separation of workloads

## Architecture

```
MySQL (Aurora RDS)
        │
        ▼
AWS Glue (Extract)
        │
        ▼
Amazon S3 (Raw Layer)
        │
        ▼
AWS Glue (Load)
        │
        ▼
Amazon Redshift (Raw → Staging → Analytics)
        │
        ▼
BI / Analytics / Reporting
```

---

## AWS Services Used

 Service and  Role 

| **Amazon Aurora MySQL (RDS)** | Source system storing operational data (e.g. apartments, user activity, events) |
| **Amazon S3** | Data lake storage for raw data ingestion (landing zone) |
| **AWS Glue** | Managed ETL service for extracting and loading data between sources |
| **Amazon Redshift** | Data warehouse for staging transformations, analytical queries, and reporting |
| **AWS Secrets Manager** | Secure storage of database credentials (MySQL & Redshift) |
| **AWS Step Functions** | Orchestration of pipeline stages (extract → load → transform) |
| **Amazon EventBridge** | Scheduling pipeline execution |
| **Amazon DynamoDB** | Maintains state for incremental loading (e.g. last extracted timestamp) |
| **AWS IAM** | Role-based access control for secure service interaction |

---

##  Pipeline Flow

### 1.  (MySQL → S3)
- Data is extracted from Aurora MySQL using AWS Glue
- Incremental logic is applied using a timestamp column
- Extracted data is stored in S3 as raw files

### 2. Raw Layer (S3)
- Stores unprocessed source data
- Acts as a single source of truth
- Enables reprocessing if needed

### 3. Load (S3 → Redshift)
- Data is loaded into Redshift using the `COPY` command
- Stored in raw schema tables

### 4. Transformation (Inside Redshift)

Data is processed through three layers:

```
Raw → Staging → Analytics
```

Key transformations include:
- Deduplication using window functions
- Incremental upserts using `MERGE`
- Data type standardisation
- Aggregations for reporting

### 5. Serving Layer (Analytics)
Final tables are optimised for dashboards, reporting, and business insights.

---

## Key Features

| Feature | Description |
|---|---|
| **Incremental Data Processing** | Uses DynamoDB to track last extracted values 
| **Scalable ELT Architecture** | Transformation happens inside Redshift |
| **Batch Processing via Glue** | Efficient handling of large datasets |
| **Secure Credential Management** | Secrets stored in AWS Secrets Manager |
| **Orchestrated Workflow** | Step Functions manage execution flow |
| **Decoupled Storage & Compute** | S3 for storage, Redshift for compute |

---

##  Example Use Cases

- User activity tracking
- Operational performance analysis
- Cost and usage reporting
- Event-based analytics

---

##  Data Model (Redshift)

### Raw Layer
```
raw_zone.apartments
raw_zone.apartment_attributes
raw_zone.apartment_viewings
```

### Staging Layer
Deduplicated and cleaned datasets derived from the raw layer.

### Analytics Layer
Aggregated and business-ready tables optimised for reporting.

---

## Technologies Used

- **Python** — Glue scripts and ETL logic
- **SQL** — Redshift transformations, `MERGE`, aggregations
- **AWS SDK** — boto3
- **Glue python shell** — via AWS Glue

---

## Best Practices Applied

- Batch loading using S3 + `COPY` (avoids row-by-row inserts)
- Use of staging tables for safe transformation
- Window functions for deduplication
- Separation of storage and compute layers
- Modular pipeline design for scalability

---

##  How to Run

1. Configure AWS credentials and IAM roles give glue permission to s3, cloudwatch and dynamo
2. Store database credentials in AWS Secrets Manager
3. Deploy Glue jobs for extraction and loading
4. Configure Step Functions workflow
5. Schedule pipeline using EventBridge
6. Monitor execution via CloudWatch