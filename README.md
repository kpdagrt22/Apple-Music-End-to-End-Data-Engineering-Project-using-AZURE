# Azure Lakehouse Platform — End-to-End Data Engineering System

This repository documents an enterprise-aligned data engineering implementation on Azure.  
The design reflects production lakehouse standards: modular ingestion, governed storage, declarative transformations, metadata-driven control, and CI/CD-based deployment of pipelines.

---

## 1. Architectural Overview

**Pattern:** Medallion Architecture (Bronze → Silver → Gold) on Delta Lake  
**Stack:** Azure SQL DB, Azure Data Factory, ADLS Gen2, Databricks (PySpark, Autoloader, DLT), Unity Catalog, Synapse, Power BI, GitHub, Key Vault, Managed Identities

**Flow**
1. Azure SQL DB → ADF → Bronze (raw, incremental, CDC-aware)  
2. Bronze → Databricks (PySpark) → Silver (cleaned, normalized, SCD-2 ready)  
3. Silver → Delta Live Tables → Gold (curated analytical models)  
4. Gold → Synapse and Power BI for consumption

The system enforces schema stability, lineage, auditability, and reproducibility across all layers.

---

## 2. Design Principles

**Metadata-Driven Execution**  
Pipeline behavior controlled entirely through JSON configuration—no per-table branching. Enables horizontal scaling across domains without pipeline rewrites.

**Incremental + Idempotent Loads**  
ADF pipelines maintain CDC watermarks per entity. Delta Lake merge logic in Silver supports safe reprocessing and backfills.

**Governed Lakehouse**  
Unity Catalog governs all entities. Three-level namespace ensures secure, portable data access.

**Separation of Concerns**  
ADF handles data movement.  
Databricks handles compute and transformation.  
DLT handles declarative ETL and quality constraints.  
Synapse delivers warehouse querying.

**Reproducibility & Versioning**  
ADF artifacts stored as ARM templates.  
Notebooks and DLT definitions versioned in Git.  
Databricks Asset Bundles manage environment deployments.

---
## 3. Repository Structure
```
/
├── adf/
│ ├── pipelines/
│ ├── datasets/
│ ├── linked_services/
│ └── arm_templates/
│
├── databricks/
│ ├── notebooks/
│ │ ├── 01_bronze_to_silver.py
│ │ ├── 02_silver_to_gold.py
│ │ ├── 03_metadata_driver.py
│ │ └── dlt_pipeline.py
│ ├── utils/
│ │ ├── scd2_merge.py
│ │ ├── cdc_watermark.py
│ │ └── autoloader_stream.py
│ ├── bundles/
│ └── jobs/
│
├── metadata/
│ ├── table_config.json
│ └── cdc_rules.json
│
├── sql/
│ ├── create_tables.sql
│ └── seed_data.sql
│
├── docs/
│ ├── architecture_diagram.png
│ ├── lineage.png
│ ├── data_model_star_schema.png
│ └── expectations.md
│
└── README.md

```
---

## 4. Pipeline Mechanics

### Bronze Ingestion
- ADF pipeline loops through metadata-defined entities.  
- Dynamic SQL resolves schema, table, CDC column, and watermark.  
- Output written to ADLS Bronze in Delta/Parquet.  
- Watermarks updated atomically to maintain incrementality.

### Silver Transformation
- Databricks enforces schema alignment, nullability, and data hygiene.  
- Autoloader supports scalable streaming ingestion.  
- SCD-2 implemented with reusable merge utilities.  
- Silver tables form normalized, business-aligned datasets.

### Gold Serving
- Delta Live Tables defines all transformations declaratively.  
- Expectations enforce data quality constraints.  
- Gold layer contains curated analytical models (star schema).

---

## 5. Data Model

### Dimensions
- dim_user  
- dim_artist  
- dim_track  
- dim_date  

### Fact
- fact_stream (grain: user × track × timestamp)

### Gold Aggregates
- artist_agg  
- daily_stream_summary  
- user_engagement_metrics

---

## 6. Operational Characteristics

**Access Control**  
- Managed Identity for Databricks–Storage.  
- Unity Catalog for privilege management and auditing.

**Observability**  
- ADF run logs for ingestion.  
- DLT lineage + quality logs.  
- Delta transaction logs for full audit history.

**Deployment**  
- All ADF assets, notebooks, and DLT pipelines versioned in Git.  
- Asset Bundles manage environment promotion.

---

## 7. Competencies Demonstrated

- Azure Data Factory (dynamic pipelines, CDC, metadata-driven patterns)  
- Databricks (PySpark ETL, Autoloader, Delta Lake, SCD-2, streaming, DLT)  
- Governance (Unity Catalog, RBAC, lineage)  
- Architecture (lakehouse design, medallion layering)  
- CI/CD (GitHub, ARM templates, Asset Bundles)  
- Data Modeling (star schema, dimensional modeling)  
- Analytics Integration (Synapse + Power BI)

---

## 8. Execution Sequence

1. Deploy Azure resources (RG, ADLS, SQL DB, ADF, Databricks, UC).  
2. Load operational schema via SQL scripts.  
3. Publish and validate Bronze ingestion pipelines.  
4. Run transformation notebooks to generate Silver layer.  
5. Deploy Gold transformations using DLT.  
6. Publish curated tables to Synapse / Power BI.

---

## 9. Future Enhancements

- Add Great Expectations for contract-based validation.  
- Enable event-driven ingestion via Event Grid.  
- Implement Databricks cluster policies + cost governance.  
- Extend monitoring via Azure Monitor + Log Analytics.





