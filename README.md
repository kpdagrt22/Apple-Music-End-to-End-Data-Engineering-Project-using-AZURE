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

The system enforces schema stability, lineage, auditability, and reproducibility across all layers.[conversation_history:0]

### 1.1 ADF CDC + Watermark Logic

This diagram shows the CDC lookup, current timestamp variable, copy activity to Data Lake, and conditional update of CDC metadata when new rows exist.[conversation_history:0]  
![ADF CDC and Watermark Flow](assets/1.png)

### 1.2 Metadata-Driven ForEach Orchestration

This diagram illustrates the metadata-driven `ForEach` loop that executes CDC extraction, copy to Data Lake, and conditional logic per entity, followed by a web activity for operational alerts.[conversation_history:0]  
![ADF ForEach Loop with Alerts](assets/2.png)

### 1.3 Incremental Copy Pipeline (Focused View)

This view focuses on the core incremental copy pipeline from CDC lookup and timestamp capture to the SQL-to-Data Lake copy and downstream conditional check for new data.[conversation_history:0]  
![ADF Incremental Copy Focused](assets/3.png)

### 1.4 Incremental Copy Pipeline (Wide View)

This wide layout presents the same incremental pattern as a full pipeline canvas, emphasizing the end-to-end wiring between CDC lookup, variable setting, copy activity, and conditional branching.[conversation_history:0]  
![ADF Incremental Copy Wide](assets/4.jpg)

---
