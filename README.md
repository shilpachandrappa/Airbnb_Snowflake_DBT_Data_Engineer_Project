# Airbnb_Snowflake_DBT_Data_Engineer_Project

🏠 Airbnb End-to-End Data Engineering Project

## 📋 Overview

This repository features a production-grade, end-to-end data engineering pipeline built to process, transform, and analyze enterprise Airbnb dataset distributions. By combining the elastic scalability of **Snowflake**, the modular orchestration of **dbt (Data Build Tool)**, and cloud storage via **AWS**, this solution demonstrates modern best practices in cloud data warehousing.

The core pipeline ingests property listings, reservation bookings, and host profile data, structuring them through a decoupled **Medallion Architecture** (Bronze → Silver → Gold). Key highlights of the implementation include programmatic incremental data loading, fully automated data quality testing, and robust historical auditing via Slowly Changing Dimensions (SCD Type 2) to build highly optimized, BI-ready data assets.

---

## 🏗️ Architecture & Core Data Flow

The architecture decouples storage from compute, utilizing AWS for durable file staging and Snowflake as the centralized analytical engine. dbt drives the transformation layer, managing dependencies and data integrity.

```text
    ┌───────────────┐      ┌────────────┐      ┌─────────────────────────┐
    │  Source Data  │ ───> │   AWS S3   │ ───> │ Snowflake Staging Area  │
    │  (Raw CSVs)   │      │ (Staging)  │      │  (External / Int Stage) │
    └───────────────┘      └────────────┘      └─────────────────────────┘
                                                            │
                                                            ▼
                                               ┌─────────────────────────┐
                                               │   🥉 BRONZE LAYER       │
                                               │   (Raw Source Tables)   │
                                               └─────────────────────────┘
                                                            │
                                                            ▼
                                               ┌─────────────────────────┐
                                               │   🥈 SILVER LAYER       │
                                               │  (Cleaned & Conformed)  │
                                               └─────────────────────────┘
                                                            │
                                                            ▼
                                               ┌─────────────────────────┐
                                               │   🥇 GOLD LAYER         │
                                               │  (Analytics & OBT/Fact) │
                                               └─────────────────────────┘
```
🛠️ Technology Stack

| Layer | Technology | Purpose |
| :--- | :--- | :--- |
| **Cloud Data Warehouse** | **Snowflake** | Centralized compute and storage for enterprise analytics |
| **Transformation Layer**| **dbt Core** | Orchestration, modular SQL engineering, and DAG management |
| **Cloud Storage** | **AWS S3** | Secure staging area for raw data ingestion |
| **Language** | **Python 3.12+** | Pipeline execution scripting and dependency management |
| **Version Control** | **Git / GitHub** | Code collaboration and production release tracking |

---

## ⚙️ Advanced dbt Features Implemented

This project demonstrates production-grade analytics engineering patterns using dbt core functionalities:

*   **⚡ Incremental Models:** Optimized data processing by processing only new or modified records, reducing Snowflake compute costs.
*   **🕒 Snapshots (SCD Type 2):** Captured historical changes over time for dimensions (`bookings`, `hosts`, `listings`) to maintain an accurate audit trail.
*   **🧩 Custom Macros:** Built reusable DRY (*Don't Repeat Yourself*) SQL functions for string utilities, data masking/trimming, and dynamic schema naming.
*   **🔥 Jinja Templating:** Dynamic SQL generation using `if-else` control flows and `for` loops directly inside models and analyses.
*   **🛡️ Robust Testing & Docs:** Automated data quality enforcement using custom and built-in 

## 📊 Data Architecture & Modeling
    This project implements a modular **Medallion Architecture** paired with **Slowly Changing Dimensions (SCD Type 2)** to transform raw, siloed source data into high-performance, analytics-ready datasets within Snowflake.

```

📥 [Raw CSVs / S3]
│
▼
🥉 [BRONZE]  Minimal transformation, schema enforcement
│
▼
🥈 [SILVER]  Deduplication, data cleaning, type casting, standardization
│
▼
🥇 [GOLD]    Business logic, dimensional modeling (Fact/Dimension), OBT

```
### 🧱 The Medallion Layering Strategy

#### 🥉 Bronze Layer (Raw Data / Ingestion)
The entry point for our data pipeline. Models in this layer mirror the source structure exactly, acting as a historical record of ingestion with zero destructive transformations.
* **`bronze_bookings`**: Direct ingestion of raw booking transactions, maintaining the original operational audit trail.
* **`bronze_hosts`**: Raw host profile metadata extracted from the source system.
* **`bronze_listings`**: Unfiltered property listing details containing unstructured or semi-structured raw text fields.

#### 🥈 Silver Layer (Cleaned & Conformed / Integration)
The data cleansing and standardization zone. Here, we apply enterprise data quality rules to prepare data for downstream relational structures.
* **`silver_bookings`**: Validated and conformed transaction records with uniform timestamp formatting, explicit data type casting, and standard handling of null values.
* **`silver_hosts`**: Enriched host profiles featuring unified names, standardized contact formats, and pre-calculated data quality metrics.
* **`silver_listings`**: Standardized property metadata featuring cleaned currency fields, uniform location tracking, and price-tier categorization logic applied via custom macros.

#### 🥇 Gold Layer (Business & Analytics-Ready)
The delivery layer optimized for business intelligence, reporting, and ad-hoc data science queries. Data is modeled for both traditional Kimball dimensional architectures and modern flat reporting structures.
* **`fact`**: A high-performance fact table built for star-schema dimensional modeling, containing granular, numeric measurement events and foreign keys mapping to our dimensions.
* **`obt` (One Big Table)**: A fully denormalized, wide dataset combining `bookings`, `listings`, and `hosts`. By pre-joining these layers, we eliminate costly runtime `JOIN` operations inside BI tools like Tableau or Looker Studio, drastically cutting Snowflake compute usage for end-users.
* **`ephemeral/` (Intermediate Models)**: Utilizes dbt's `ephemeral` materialization pattern to handle complex multi-stage CTE transformations. These temporary views exist only during compile time, keeping the Snowflake schema clean and free of intermediate clutter.


### 🕒 Snapshot Tracking (SCD Type 2)
To preserve data lineage and enable accurate point-in-time analysis, the pipeline uses dbt snapshots to manage **Slowly Changing Dimensions (SCD Type 2)**. This captures mutation history without overwriting existing rows, stamping records with `dbt_valid_from` and `dbt_valid_to` fields.

| Snapshot Dimension | Business Tracking Purpose |
| :--- | :--- |
| **`dim_bookings`** | Tracks historical booking alterations, state modifications, and status changes. |
| **`dim_hosts`** | Captures changes in host verification statuses, contact updates, and evolving quality tier rankings. |
| **`dim_listings`** | Documents price fluctuations, structural updates, and availability changes over the lifetime of a property. |

```


📁 Project Structure

This repository is structured as an end-to-end analytics engineering pipeline utilizing AWS, dbt, and Snowflake. Below is the directory layout and description of key components:

```text
AWS_DBT_Snowflake/
├── README.md                          # This file
├── pyproject.toml                     # Python dependencies
├── main.py                            # Main execution script
│
├── SourceData/                        # Raw CSV data files
│   ├── bookings.csv
│   ├── hosts.csv
│   └── listings.csv
│
├── DDL/                               # Database schema definitions
│   ├── ddl.sql                        # Table creation scripts
│   └── resources.sql
│
└── aws_dbt_snowflake_project/         # Main dbt project
    ├── dbt_project.yml                # dbt project configuration
    ├── ExampleProfiles.yml            # Snowflake connection profile
    │
    ├── models/                        # dbt models
    │   ├── sources/
    │   │   └── sources.yml            # Source definitions
    │   ├── bronze/                    # Raw data layer
    │   │   ├── bronze_bookings.sql
    │   │   ├── bronze_hosts.sql
    │   │   └── bronze_listings.sql
    │   ├── silver/                    # Cleaned data layer
    │   │   ├── silver_bookings.sql
    │   │   ├── silver_hosts.sql
    │   │   └── silver_listings.sql
    │   └── gold/                      # Analytics layer
    │       ├── fact.sql
    │       ├── obt.sql
    │       └── ephemeral/             # Temporary models
    │           ├── bookings.sql
    │           ├── hosts.sql
    │           └── listings.sql
    │
    ├── macros/                        # Reusable SQL functions
    │   ├── generate_schema_name.sql   # Custom schema naming
    │   ├── multiply.sql               # Math operations
    │   ├── tag.sql                    # Categorization logic
    │   └── trimmer.sql                # String utilities
    │
    ├── analyses/                      # Ad-hoc analysis queries
    │   ├── explore.sql
    │   ├── if_else.sql
    │   └── loop.sql
    │
    ├── snapshots/                     # SCD Type 2 configurations
    │   ├── dim_bookings.yml
    │   ├── dim_hosts.yml
    │   └── dim_listings.yml
    │
    ├── tests/                         # Data quality tests
    │   └── source_tests.sql
    │
    └── seeds/                         # Static reference data
```


## 🎯 Key Features

###1. Incremental Loading
    Bronze and silver models utilize dbt's incremental materialization pattern to process only new or updated records. This drastically reduces Snowflake compute footprint by eliminating full table scans.

```sql
{{ config(
    materialized='incremental',
    unique_key='id'
) }}

SELECT * FROM {{ ref('bronze_bookings') }}
{% if is_incremental() %}
  WHERE updated_at > (SELECT COALESCE(MAX(updated_at), '1900-01-01') FROM {{ this }})
{% endif %}
```
###2. Custom Macros
    To maintain a DRY (Don't Repeat Yourself) codebase, reusable business logic is encapsulated inside custom dbt macros. For example, the tag() macro dynamically categorizes listing price tiers:
```
-- Application within models:
{{ tag('CAST(PRICE_PER_NIGHT AS INT)') }} AS PRICE_PER_NIGHT_TAG
---
````
###3. Dynamic SQL Generation (Jinja Loops)
    The OBT (One Big Table) and downstream analytical structures leverage Jinja control flows and loops. Instead of hardcoding repetitive `JOIN` or `SELECT` statements, the code dynamically generates structural columns:

```sql
{% set models = ['bookings', 'hosts', 'listings'] %}

SELECT 
  {% for model in models %}
    {{ model }}.* {% if not loop.last %},{% endif %}
  {% endfor %}
FROM {{ ref('fact_bookings') }}
```
###4. Slowly Changing Dimensions (SCD Type 2)
    SCD Type 2 Source mutation tracking is fully automated via dbt snapshots. The pipeline tracks historical changes over time with timestamp-based monitoring:dbt_valid_from and dbt_valid_to fields are automatically managed.Historical data is completely preserved, enabling seamless point-in-time analytical auditing.

###5.Multi-Layer Schema Organization
    Multi-Layer Schema OrganizationTo maintain rigid data governance and clear separation of concerns, custom schema configurations dynamically route objects to dedicated layers within Snowflake:
🥉 Bronze Models $\rightarrow$ AIRBNB.BRONZE.*
🥈 Silver Models $\rightarrow$ AIRBNB.SILVER.*
🥇 Gold Models $\rightarrow$ AIRBNB.GOLD.*


## 📈 Data Quality & Governance

### 🛡️ Testing Strategy
We enforce rigorous data quality standards at every layer of the pipeline using a combination of built-in dbt expectations and custom data tests:
* **Source Data Validation:** Schema enforcement and freshness checks to ensure raw data meets structural expectations before transformation.
* **Primary Key Constraints:** Automated `unique` and `not_null` validation across all staging, dimension, and fact grains.
* **Referential Integrity:** Operational tracking using `relationships` tests to validate foreign key mapping between facts and dimensions.
* **Custom Business Rules:** Bespoke singular and generic data tests to capture domain-specific data anomalies (e.g., ensuring checkout dates are strictly greater than check-in dates).

### 🔍 Automated Data Lineage
Data lineage is automatically generated, tracked, and maintained by dbt. This compile-time dependency tracking provides:
* **Upstream Traceability:** Full visibility into raw source dependencies for every downstream data asset.
* **Impact Analysis:** Clear insight into downstream impacts prior to making structural schema changes.
* **Directed Acyclic Graphs (DAGs):** End-to-end operational visibility from initial raw staging areas to final Gold BI layers.

---

## 🔐 Security & Engineering Best Practices

### 🔑 Credentials & Access Management
* **Zero-Credential Repositories:** Local and production profiles (`profiles.yml`) utilize environment variables (`ENV_VAR`) for authentication. Hardcoded passwords or private keys are strictly prohibited from version control.
* **Snowflake RBAC:** Strict alignment with Snowflake’s Role-Based Access Control (RBAC) hierarchy, ensuring least-privilege access across `transformer`, `loader`, and `reporter` roles.
* **Data Masking:** Sensitive data or Personally Identifiable Information (PII) is hashed or obfuscated at the Silver layer using custom masking macros.

### 💻 Code Quality & CI
* **Deterministic Formatting:** Code style rules are strictly enforced using `sqlfmt` to maintain uniform SQL readability across all models.
* **Git-Driven Workflow:** Production deployments require feature branch isolation, explicit peer code reviews, and passing integration test suites.

* ## ⚡ Performance Optimization

To maintain highly efficient compute costs and lower execution latency within Snowflake, the project applies specific materialization strategies:
* **Incremental Materialization:** Heavy transaction tables are built incrementally to process only new or mutated rows, eliminating full table scans.
* **Ephemeral Sub-Queries:** Lightweight intermediate transformations use `ephemeral` materialization, preventing database clutter and saving storage costs by compiling down to nested CTEs.
* **Micro-Partitioning & Clustering:** Gold-layer tables leverage optimal query filtering keys, ensuring Snowflake utilizes metadata partition pruning for analytical queries.

---

## 🚀 Future Enhancements

- [ ] **Slim CI/CD Pipelines:** Implement GitHub Actions to run `dbt source freshness` and execute modified models using state tracking (`dbt run --select state:modified+`).
- [ ] **Observability & Alerting:** Integrate data observability tools (e.g., Elementary or Monte Carlo) to push real-time anomaly alerts to Slack/Teams.
- [ ] **PII Masking Policies:** Deploy native Snowflake dynamic data masking policies managed directly through dbt pre-hooks.
- [ ] **BI Tool Integration:** Build a semantic layer mapping the One Big Table (OBT) directly to Looker Studio or Tableau dashboard environments.

---

## 📚 Additional Resources

* [Official dbt Core Documentation](https://docs.getdbt.com/)
* [Snowflake Architecture Guide](https://docs.snowflake.com/)
* [dbt Labs Analytics Engineering Best Practices](https://docs.getdbt.com/guides/best-practices)
