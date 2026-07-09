# Azure Data Engineer Interview — Last-Minute Prep Guide

> **How to use this:** Skim the **Theory** for crisp definitions, read the **Code/Config** to see it in practice, and rehearse the **Interview Q&A** out loud. Bold lines are the one-liners interviewers want to hear. Covers the full Azure data stack — ADF, Delta Lake, Medallion, Microsoft Fabric, Synapse, ADLS, security. Built for a 2–4 hour final revision.

---

## Table of Contents

1. [Azure Data Engineering Landscape](#1-azure-data-engineering-landscape)
2. [Azure Data Lake Storage (ADLS Gen2)](#2-azure-data-lake-storage-adls-gen2)
3. [Azure Data Factory — Core Concepts](#3-azure-data-factory--core-concepts)
4. [Integration Runtime (IR)](#4-integration-runtime-ir)
5. [ADF Activities, Pipelines & Data Flows](#5-adf-activities-pipelines--data-flows)
6. [Triggers & Scheduling](#6-triggers--scheduling)
7. [Incremental Load / Watermark Pattern](#7-incremental-load--watermark-pattern)
8. [Delta Lake](#8-delta-lake)
9. [Medallion Architecture](#9-medallion-architecture)
10. [Microsoft Fabric](#10-microsoft-fabric)
11. [Azure Synapse Analytics](#11-azure-synapse-analytics)
12. [Azure SQL & Data Modeling / Warehousing](#12-azure-sql--data-modeling--warehousing)
13. [Security, Governance & Secrets](#13-security-governance--secrets)
14. [CI/CD & DevOps for Azure Data](#14-cicd--devops-for-azure-data)
15. [Monitoring, Error Handling & Cost Optimization](#15-monitoring-error-handling--cost-optimization)
16. [Scenario / Design Questions (with answers)](#16-scenario--design-questions)
17. [Common Code / Config Snippets](#17-common-code--config-snippets)
18. [Tricky / Gotcha Questions](#18-tricky--gotcha-questions)
19. [Rapid-Fire One-Liners](#19-rapid-fire-one-liners)
20. [Last-Minute Checklist](#20-last-minute-checklist)

---

## 1. Azure Data Engineering Landscape

### Theory

A typical **Azure modern data platform** flows:

**Sources** (on-prem DBs, APIs, files, SaaS) → **Ingest/Orchestrate** (Azure Data Factory) → **Store** (ADLS Gen2 as the data lake) → **Transform** (Azure Databricks / Synapse Spark / Data Flows) → **Serve** (Synapse SQL / Azure SQL / Fabric Warehouse) → **Consume** (Power BI, ML).

| Service | Role |
|---|---|
| **Azure Data Factory (ADF)** | Cloud ETL/ELT **orchestration** & data movement (PaaS) |
| **ADLS Gen2** | Scalable, hierarchical **data lake storage** |
| **Azure Databricks** | Spark-based **transformation**, ML, lakehouse |
| **Azure Synapse Analytics** | Unified analytics: SQL pools + Spark + pipelines |
| **Microsoft Fabric** | **SaaS** unified analytics platform (OneLake) |
| **Azure SQL Database** | Managed relational DB (OLTP) |
| **Power BI** | BI / visualization |
| **Azure Key Vault** | Secrets / credentials |
| **Microsoft Entra ID (Azure AD)** | Identity & access |

**ETL vs ELT:** ETL transforms **before** loading (traditional); **ELT** loads raw data first, then transforms in the powerful target (lake/warehouse) — the modern cloud pattern, ideal for the medallion architecture.

### Interview Q&A

**Q: Describe a typical Azure data pipeline architecture.**
Sources are ingested by **ADF** into **ADLS Gen2** (Bronze), transformed with **Databricks/Synapse Spark** through **Silver and Gold** (medallion) Delta layers, then served via **Synapse/Fabric Warehouse** to **Power BI**. Security via **Entra ID + Key Vault**, orchestrated and scheduled by ADF.

**Q: ETL vs ELT — which and why on Azure?**
**ELT** is preferred in the cloud: load raw data into ADLS/lakehouse first, then transform using scalable compute (Spark/SQL pools). It's cheaper, more flexible, supports reprocessing, and fits the medallion pattern. ETL still suits cases needing transformation before landing (e.g., PII masking pre-storage).

**Q: PaaS vs SaaS in this stack?**
**ADF and Synapse are PaaS** (you configure/manage some infrastructure). **Microsoft Fabric is SaaS** — fully managed, unified, with capacity-based billing and OneLake as the single store.

---

## 2. Azure Data Lake Storage (ADLS Gen2)

### Theory

**ADLS Gen2** = Azure Blob Storage **+ a hierarchical namespace (HNS)** — real directories/folders, atomic directory operations, and POSIX-like ACLs. It's the standard data lake for analytics.

- **HNS** makes folder operations (rename/delete) fast and atomic (vs flat blob namespace).
- **Access:** account keys/SAS (legacy), **service principal + OAuth**, **managed identity**, or **Unity Catalog external locations** (preferred in Databricks).
- **Formats:** prefer **Parquet/Delta** (columnar, compressed, schema, predicate pushdown) over CSV/JSON for analytics.
- **Partitioning:** organize files by date/business key for partition pruning.
- **Lifecycle management:** auto-tier to cool/archive or delete old data.

### Interview Q&A

**Q: ADLS Gen2 vs Blob Storage?**
ADLS Gen2 is Blob Storage **with a hierarchical namespace**, giving real directories, **atomic** folder operations, fine-grained **POSIX ACLs**, and optimized analytics performance. Blob is a flat object store. Gen2 is built for big-data workloads.

**Q: How do you secure access to ADLS from Databricks/ADF?**
Preferably **managed identity** or a **service principal with OAuth**, with credentials stored in **Key Vault**. In Unity Catalog, use **storage credentials + external locations**. Avoid account keys/SAS in code.

**Q: Why Parquet/Delta over CSV?**
They're **columnar and compressed** with embedded schema, enabling **column pruning** and **predicate pushdown** — far less I/O than row-based CSV/JSON.

**Q: How do you organize data in the lake?**
By **medallion zones** (raw/bronze, silver, gold) and **partitioned folders** (e.g., `/year=/month=/day=`), with consistent naming and Parquet/Delta formats.

---

## 3. Azure Data Factory — Core Concepts

### Theory

**Azure Data Factory (ADF)** is a **cloud-based, serverless ETL/ELT and orchestration** service (PaaS). It moves and transforms data via pipelines without managing servers.

**Core building blocks (know all of these):**

- **Pipeline** — a logical grouping of **activities** that perform a task.
- **Activity** — a single step (Copy, Data Flow, Databricks notebook, Lookup, ForEach, etc.).
- **Linked Service** — a **connection string** to a data source/compute (like a DB connection definition).
- **Dataset** — a **named view/pointer** to the data structure within a linked service (table/file).
- **Integration Runtime (IR)** — the **compute** that runs the activity (see §4).
- **Trigger** — what starts a pipeline (schedule/event/manual).

**Mental model:** *Linked Service = where & how to connect; Dataset = what data; Activity = what to do; Pipeline = the workflow; IR = the engine.*

### Code (ADF JSON)

```json
// Linked Service (ADLS Gen2)
{
  "name": "ls_adls",
  "properties": {
    "type": "AzureBlobFS",
    "typeProperties": { "url": "https://acct.dfs.core.windows.net" }
  }
}
```

```json
// Copy Activity (inside a pipeline)
{
  "name": "CopyToBronze",
  "type": "Copy",
  "inputs": [{ "referenceName": "ds_source_sql" }],
  "outputs": [{ "referenceName": "ds_bronze_parquet" }],
  "typeProperties": {
    "source": { "type": "AzureSqlSource" },
    "sink": { "type": "ParquetSink" }
  }
}
```

### Interview Q&A

**Q: What is Azure Data Factory?**
A **serverless, cloud ETL/ELT and orchestration** service for building data-driven pipelines that **move and transform** data across on-prem and cloud sources, with scheduling, monitoring, and CI/CD support.

**Q: Linked Service vs Dataset?**
A **Linked Service** is the **connection** (where + credentials, like a connection string). A **Dataset** is a **pointer to specific data** (a table/file) *within* that linked service. One linked service can back many datasets.

**Q: What is the Copy Activity?**
ADF's core **data-movement** activity — it copies data from a source to a sink, with optional format conversion, mapping, and built-in **fault tolerance/staging**. It runs on an Integration Runtime.

**Q: Is ADF code or GUI?**
Primarily a **low-code GUI** (with underlying JSON), but supports parameterization, expressions, and **CI/CD via Git + ARM templates**.

---

## 4. Integration Runtime (IR)

### Theory

**Integration Runtime (IR)** is the **compute infrastructure** ADF uses for **data movement, transformation (Data Flows), and activity dispatch**. Three types:

| IR Type | Use Case |
|---|---|
| **Azure IR** | Fully managed; cloud-to-cloud data movement & Data Flow execution across Azure/public sources |
| **Self-hosted IR (SHIR)** | Connects to **on-premises** or **private network/VNet** sources securely; you install it on a local/VM machine |
| **Azure-SSIS IR** | Lift-and-shift **run existing SSIS packages** in ADF |

### Interview Q&A

**Q: What is an Integration Runtime and what are the types?**
The **compute** ADF uses to move/transform data and dispatch activities. **Azure IR** for cloud sources, **Self-hosted IR** for on-prem/private networks, **Azure-SSIS IR** to run legacy SSIS packages.

**Q: When do you need a Self-hosted IR?**
When connecting to **on-premises** data sources, data behind a **firewall/private VNet**, or any source not publicly reachable — the SHIR acts as a secure gateway installed in your network.

**Q: Can one Self-hosted IR be shared?**
Yes — a single SHIR can be **shared across multiple data factories** and can be set up with **multiple nodes** for high availability and scale-out.

---

## 5. ADF Activities, Pipelines & Data Flows

### Theory

**Activity categories:**

- **Data movement:** Copy Activity.
- **Data transformation:** **Mapping Data Flows** (visual, code-free Spark transforms), Databricks notebook/jar/python, Synapse notebook/Spark, Stored Procedure, HDInsight.
- **Control flow:** **Lookup**, **Get Metadata**, **ForEach** (iterate), **If Condition**, **Switch**, **Until**, **Set/Append Variable**, **Execute Pipeline**, **Wait**, **Web/Webhook**.

**Mapping Data Flow vs Copy vs Databricks (key decision):**

- **Copy Activity** — pure **movement** (and simple format conversion); no complex transforms.
- **Mapping Data Flow** — **visual, code-free transformations** that run on a managed Spark cluster (ADF spins it up). Good when you want transforms without writing code.
- **Databricks/Synapse notebook** — **code-first** (PySpark/SQL) for complex/large-scale transforms; most control and performance.

**Parameters vs Variables:** **parameters** are passed in at runtime (external inputs, read-only inside); **variables** are mutable within a pipeline run (`Set/Append Variable`).

### Interview Q&A

**Q: Copy Activity vs Mapping Data Flow vs calling Databricks?**
**Copy** moves data (no real transforms). **Mapping Data Flow** does **visual, code-free** transformations on a managed Spark cluster. **Databricks/Synapse notebooks** give **code-first**, large-scale, fully customizable transforms. Choose based on complexity, team skills, and control needed.

**Q: How does ForEach work and is it parallel?**
`ForEach` iterates over an array (often from a **Lookup** or **Get Metadata**), running inner activities per item. It runs **in parallel by default** (configurable batch count); set **sequential** when order/serialization matters.

**Q: How do you make a pipeline reusable?**
**Parameterize** linked services, datasets, and pipelines (e.g., table name, file path, date), then drive them via **Lookup + ForEach** — a single metadata-driven pipeline can handle many tables.

**Q: What is the Lookup activity used for?**
To **read a value or set of rows** (e.g., a config/control table or the last watermark) and pass it to downstream activities — central to **metadata-driven** and **incremental** patterns.

---

## 6. Triggers & Scheduling

### Theory

Three trigger types:

- **Schedule trigger** — runs on a wall-clock schedule (e.g., daily 2 AM). Many-to-many with pipelines.
- **Tumbling window trigger** — fixed-size, **non-overlapping**, contiguous time windows; supports **dependencies, backfill, and retry** with windows (great for incremental/historical loads). One-to-one with a pipeline.
- **Event-based trigger** — fires on **blob/storage events** (file created/deleted) — good for arrival-driven ingestion. (Also: **custom event** via Event Grid.)

### Interview Q&A

**Q: Schedule vs tumbling window trigger?**
A **schedule** trigger fires on a recurring clock time and can map to many pipelines. A **tumbling window** trigger runs in **contiguous, non-overlapping** time windows and supports **backfill, window dependencies, and per-window retries** — ideal for incremental/time-sliced loads.

**Q: How do you trigger a pipeline when a file arrives?**
Use an **event-based (storage event) trigger** that listens for blob-created events in ADLS via **Event Grid** — the pipeline starts automatically when the file lands.

**Q: How do you backfill historical data?**
Use a **tumbling window** trigger with a past start date — it generates and runs a window per interval, honoring dependencies, so historical periods are processed in order.

---

## 7. Incremental Load / Watermark Pattern

### Theory

**Don't reload everything nightly** — load only **new/changed** rows. The classic **watermark pattern**:

1. Store the **last loaded value** (max `LastModifiedDate` or `ID`) in a **control/watermark table**.
2. **Lookup** the old watermark.
3. **Copy** only rows where `column > old_watermark`.
4. Get the **new max** value and **update** the watermark table.

Alternatives: **Change Tracking / CDC** (SQL Server/Azure SQL), **Auto Loader** (`cloudFiles`) for incremental *file* ingestion in Databricks, and **delta/merge** for upserts.

### Code

```sql
-- Source query parameterized by watermark
SELECT * FROM sales
WHERE LastModifiedDate > '@{activity('LookupOldWatermark').output.firstRow.wm}';
```

```json
// ADF flow: Lookup(old wm) -> Copy(filtered) -> Lookup(new max) -> SP/Copy(update wm)
```

### Interview Q&A

**Q: How do you implement incremental loading in ADF?**
Use the **watermark pattern**: keep the last-loaded timestamp/ID in a **control table**, **Lookup** it, **filter the source** to rows greater than it during Copy, then **update the watermark** with the new max. Only deltas move each run.

**Q: What if the source has no reliable timestamp?**
Use **Change Tracking / CDC** if available, a **hash comparison** to detect changes, or full extract into a staging area then **MERGE** to apply only differences.

**Q: How do you avoid duplicates on re-runs (idempotency)?**
Make loads **idempotent**: use **MERGE/upsert** on a business key into Delta, or overwrite the affected partition. Combined with checkpoints/watermarks, a re-run won't double-insert.

**Q: How do you do upserts in the lake?**
Delta Lake **`MERGE INTO`** on a key — update matched rows, insert new ones (SCD Type 1), or expire + insert for SCD Type 2.

---

## 8. Delta Lake

### Theory

**Delta Lake** = Parquet data files **+ a transaction log (`_delta_log`)** that brings **reliability** to the lake.

**Headline features:** **ACID transactions**, **time travel** (versioning), **schema enforcement & evolution**, **MERGE/UPDATE/DELETE**, unified **batch + streaming**, and scalable metadata.

**Maintenance commands:** `OPTIMIZE` (compact small files), `ZORDER BY` (co-locate for data skipping), `VACUUM` (remove old files; default 7-day retention; breaks time travel below that), `DESCRIBE HISTORY` (versions). **Liquid Clustering** is the modern alternative to partitioning + Z-Order.

### Code

```sql
-- Time travel & rollback
SELECT * FROM sales VERSION AS OF 5;
RESTORE TABLE sales TO VERSION AS OF 5;

-- Upsert (MERGE)
MERGE INTO sales t USING updates s ON t.id = s.id
WHEN MATCHED THEN UPDATE SET *
WHEN NOT MATCHED THEN INSERT *;

-- Maintenance
OPTIMIZE sales ZORDER BY (customer_id);
VACUUM sales RETAIN 168 HOURS;
DESCRIBE HISTORY sales;
```

### Interview Q&A

**Q: Delta Lake vs Parquet?**
Delta is **Parquet + a transaction log**, adding **ACID transactions, time travel, schema enforcement/evolution, and MERGE/UPDATE/DELETE** — capabilities plain Parquet lacks.

**Q: How does Delta give ACID on a data lake?**
Via the **`_delta_log`** and **optimistic concurrency control** — each write commits atomically; readers get a consistent **snapshot**; conflicting writes retry.

**Q: What is time travel and a use case?**
Query/restore a previous table version (`VERSION/TIMESTAMP AS OF`). Used for **audits, rollback after a bad load, and reproducible ML**.

**Q: OPTIMIZE vs VACUUM?**
`OPTIMIZE` **compacts small files** (and Z-Orders for data skipping); `VACUUM` **deletes old, unreferenced files** to reclaim storage (and limits time travel afterward).

---

## 9. Medallion Architecture

### Theory

A layered Delta design refining data quality progressively:

- **Bronze (Raw):** ingest source data **as-is**, append-only, full history — auditable & reprocessable.
- **Silver (Cleansed):** **cleaned, deduplicated, schema-enforced, validated, joined/conformed** — the trusted enterprise view.
- **Gold (Curated):** **business aggregates, marts, feature tables** for BI/ML.

**Why:** clear separation of concerns, **traceable data quality**, and the ability to **rebuild downstream layers from Bronze**.

### Code

```python
from pyspark.sql import functions as F
# Bronze -> Silver: clean + dedupe
(spark.table("bronze.events")
   .dropDuplicates(["event_id"])
   .filter(F.col("user_id").isNotNull())
   .write.format("delta").mode("overwrite").saveAsTable("silver.events"))
# Silver -> Gold: aggregate
(spark.table("silver.events").groupBy("country").count()
   .write.format("delta").mode("overwrite").saveAsTable("gold.events_by_country"))
```

### Interview Q&A

**Q: Explain the medallion architecture.**
Three Delta layers: **Bronze** (raw, as-is), **Silver** (cleaned, deduped, schema-validated, conformed), **Gold** (curated aggregates for BI/ML). It improves data quality, traceability, and reprocessability.

**Q: What happens at the Silver layer specifically?**
**Deduplication, schema enforcement, type casting, null/quality handling, and joins/conforming** — turning raw Bronze into a reliable, query-ready dataset.

**Q: Why keep raw Bronze data?**
For **audit and reprocessing** — if logic changes or a bug appears, you rebuild Silver/Gold from untouched raw history without re-ingesting from source.

---

## 10. Microsoft Fabric

### Theory

**Microsoft Fabric** is an **end-to-end, SaaS unified analytics platform** that brings together data engineering, data integration, data warehousing, data science, real-time analytics, and BI — all on a single foundation called **OneLake**.

**Key concepts:**

- **OneLake** — a **single, unified, tenant-wide data lake** ("OneDrive for data"); **one copy** of data in **Delta/Parquet**, accessible by all engines. **Shortcuts** reference data in place (ADLS, S3) without copying.
- **Workloads/Experiences:** **Data Factory** (pipelines + Dataflows Gen2), **Data Engineering** (Spark, Lakehouse), **Data Warehouse** (T-SQL), **Data Science**, **Real-Time Intelligence**, **Power BI**.
- **Lakehouse vs Warehouse:** Lakehouse = Spark + files + tables (Delta); Warehouse = full **T-SQL**, transactional warehouse. Both store in OneLake as Delta.
- **Direct Lake mode** — Power BI reads Delta tables **directly from OneLake** (no import, no DirectQuery latency) — fast + fresh.
- **Capacity-based (SaaS) billing**, not per-resource.

**ADF vs Fabric Data Factory:** same lineage/engine, but ADF is **PaaS** (standalone service) while **Data Factory in Fabric is SaaS** integrated into the Fabric ecosystem (Dataflows Gen2, OneLake-native).

### Interview Q&A

**Q: What is Microsoft Fabric?**
A **SaaS, all-in-one analytics platform** unifying data integration, engineering, warehousing, data science, real-time analytics, and Power BI on **OneLake**, with capacity-based billing — reducing the need to stitch separate services.

**Q: What is OneLake?**
Fabric's **single, organization-wide data lake** (one logical lake per tenant) storing **one copy** of data in open **Delta/Parquet** format, usable by every Fabric engine. **Shortcuts** let you reference external/existing data without copying.

**Q: Lakehouse vs Warehouse in Fabric?**
A **Lakehouse** is Spark/file-oriented with Delta tables (read/write via Spark and SQL endpoint, read-only T-SQL). A **Warehouse** is a full **T-SQL transactional** warehouse (read/write SQL). Both persist to OneLake as Delta; choose by team skillset and workload.

**Q: What is Direct Lake mode?**
A Power BI storage mode that reads Delta tables **directly from OneLake** — combining **Import speed with DirectQuery freshness**, no data duplication or refresh lag.

**Q: ADF vs Data Factory in Fabric?**
Both share the same data-integration heritage. **ADF is PaaS** (a standalone Azure service); **Fabric Data Factory is SaaS** embedded in Fabric, with **Dataflows Gen2** and native OneLake integration. Migrate to Fabric when consolidating on the Fabric ecosystem.

**Q: Fabric vs Databricks — when?**
**Fabric** is a unified SaaS suite tightly integrated with Power BI/Microsoft 365, great for organizations wanting an all-in-one, low-friction platform. **Databricks** offers deeper, more mature Spark/ML/lakehouse engineering control and multi-cloud. They overlap on lakehouse but differ in openness, control, and ecosystem.

---

## 11. Azure Synapse Analytics

### Theory

**Azure Synapse Analytics** is a unified analytics service combining:

- **Dedicated SQL Pool** (formerly SQL DW) — **MPP** (massively parallel processing) data warehouse with **distribution** strategies (Hash, Round-Robin, Replicated).
- **Serverless SQL Pool** — query files in the lake **on-demand** (pay per query, T-SQL over Parquet/CSV).
- **Spark Pool** — Apache Spark for big-data transforms/ML.
- **Synapse Pipelines** — same engine as **ADF** for orchestration.
- **Synapse Link** — near-real-time analytics over operational stores (Cosmos DB, SQL).

**Distributions (dedicated pool):** **Hash** (large fact tables, even spread on a key), **Round-Robin** (staging/even load, no key), **Replicated** (small dimensions copied to all nodes).

**PolyBase / COPY INTO** — high-throughput bulk loading from the lake into the warehouse.

### Interview Q&A

**Q: What is Azure Synapse?**
A unified analytics platform with **dedicated SQL pools (MPP warehouse)**, **serverless SQL** (query the lake on demand), **Spark pools**, and **pipelines** (ADF engine) — bridging data warehousing and big-data analytics.

**Q: Dedicated vs serverless SQL pool?**
**Dedicated** is a provisioned **MPP warehouse** with reserved compute (DWUs) for predictable, high-performance workloads. **Serverless** is **on-demand T-SQL over lake files**, pay-per-query, ideal for exploration/ad-hoc without provisioning.

**Q: Explain table distributions.**
In a dedicated pool, data is spread across 60 distributions. **Hash** distributes by a column (best for large facts, minimizes shuffle on joins). **Round-Robin** spreads evenly with no key (good for staging). **Replicated** copies a small table to every node (great for small dimensions).

**Q: How do you load data into Synapse efficiently?**
Use **COPY INTO** (or PolyBase) for parallel bulk loads from ADLS, often staging as Parquet, into Round-Robin staging then inserting into Hash-distributed targets.

---

## 12. Azure SQL & Data Modeling / Warehousing

### Theory

- **Azure SQL Database** — managed **relational (OLTP)** database (PaaS); auto-patching, HA, scaling. Good for transactional/serving layers.
- **OLTP vs OLAP:** OLTP = many small transactions, normalized; OLAP = analytical, aggregations, denormalized/star schema.
- **Dimensional modeling:** **Star schema** (central fact + denormalized dimensions) vs **Snowflake** (normalized dimensions). **Fact** = measures/events; **Dimension** = descriptive context. **Grain** = level of detail of a fact row.
- **Slowly Changing Dimensions (SCD):** Type 1 (overwrite), **Type 2** (keep history with start/end dates + current flag), Type 3 (limited history columns).

### Code

```sql
-- SCD Type 2 with Delta MERGE
MERGE INTO dim_customer t
USING updates s ON t.id = s.id AND t.is_current = true
WHEN MATCHED AND t.attr <> s.attr THEN
  UPDATE SET t.is_current = false, t.end_date = current_date()
WHEN NOT MATCHED THEN
  INSERT (id, attr, is_current, start_date) VALUES (s.id, s.attr, true, current_date());
```

### Interview Q&A

**Q: Star vs snowflake schema?**
**Star** has a central **fact** table joined to **denormalized dimensions** — fewer joins, faster queries, simpler for BI. **Snowflake** normalizes dimensions into sub-tables — less redundancy but more joins. Star is usually preferred for analytics.

**Q: Fact vs dimension table?**
A **fact** table stores **measurable events/metrics** (sales amount, quantity) at a defined **grain**, with foreign keys to dimensions. A **dimension** table stores **descriptive attributes** (customer, product, date) for filtering/grouping.

**Q: Explain SCD Type 1 vs Type 2.**
**Type 1** overwrites the old value (no history). **Type 2** preserves history by adding a **new row** with versioning columns (`start_date`, `end_date`, `is_current`) — implemented via **Delta `MERGE`**.

**Q: OLTP vs OLAP?**
**OLTP** handles many small, normalized transactions (Azure SQL). **OLAP** is read-heavy analytical processing over aggregated, denormalized data (Synapse/lakehouse Gold).

---

## 13. Security, Governance & Secrets

### Theory

- **Identity:** **Microsoft Entra ID (Azure AD)** for authentication/SSO; **RBAC** for resource-level access; **managed identities** for service-to-service auth (no passwords).
- **Secrets:** store credentials/keys in **Azure Key Vault**; reference from ADF (via linked service) and Databricks (Key Vault–backed secret scope). Never hardcode.
- **Data governance:** **Unity Catalog** (Databricks) / **Microsoft Purview** (enterprise catalog, lineage, classification) for discovery, lineage, and policy.
- **Network:** VNet integration, Private Endpoints/Private Link, firewalls, NSGs.
- **Data protection:** encryption at rest (default) and in transit (TLS); column/row-level security; dynamic data masking; PII handling.

### Code

```python
# Databricks: fetch secret from Key Vault–backed scope
key = dbutils.secrets.get(scope="kv-scope", key="adls-sp-secret")
```

### Interview Q&A

**Q: How do you manage secrets in Azure data pipelines?**
Store them in **Azure Key Vault** and reference at runtime — ADF via a **Key Vault linked service**, Databricks via a **Key Vault–backed secret scope** (`dbutils.secrets.get`). Credentials are never hardcoded and are redacted in logs.

**Q: What is a managed identity and why use it?**
An **Entra ID identity** automatically managed by Azure for a service (ADF/Databricks), letting it authenticate to resources (ADLS, Key Vault) **without storing credentials** — more secure than keys/secrets.

**Q: What is Microsoft Purview?**
An enterprise **data governance** service for **cataloging, classification, lineage, and access policies** across the data estate — complements Unity Catalog for organization-wide discovery and compliance.

**Q: How do you secure data access at row/column level?**
**Row-level security** and **column masking** (in Synapse/SQL/Unity Catalog), plus RBAC and Entra ID groups — so users only see authorized data.

---

## 14. CI/CD & DevOps for Azure Data

### Theory

- **ADF CI/CD:** develop in a **Git-integrated** factory (feature branches), publish to generate **ARM templates** from the `adf_publish` branch, deploy across **dev → test → prod** via Azure DevOps/GitHub Actions, with **parameterized** linked services per environment.
- **Databricks CI/CD:** **Databricks Repos** + **Asset Bundles (DABs)**, deploy notebooks/jobs via pipelines.
- **Practices:** environment parameterization (Key Vault per env), automated tests/data validation, infrastructure-as-code (ARM/Bicep/Terraform).

### Interview Q&A

**Q: How do you implement CI/CD for ADF?**
Connect ADF to **Git**, develop on feature branches, merge to `main`, then **publish** to produce **ARM templates** (`adf_publish`). A release pipeline (Azure DevOps/GitHub Actions) deploys those templates to higher environments with **environment-specific parameters** (Key Vault, linked services).

**Q: How do you handle environment-specific configs?**
**Parameterize** linked services and pipelines and supply per-environment values (connection strings, paths, Key Vault references) at deployment via **ARM template parameters**.

**Q: How do you version-control Databricks work?**
**Databricks Repos** (Git integration) for notebooks/code and **Databricks Asset Bundles** to package and deploy jobs/pipelines through CI/CD.

---

## 15. Monitoring, Error Handling & Cost Optimization

### Theory

- **Monitoring:** ADF **Monitor** tab, **Azure Monitor / Log Analytics**, alerts; Databricks job runs & Spark UI; Synapse monitoring.
- **Error handling in ADF:** activity **retry policy + timeout**, **failure/`on-failure` paths**, `If/Until` for conditional logic, alerting via **Logic Apps/email/Teams**.
- **Reliability:** idempotent/restartable design, checkpoints, watermark-based resumability.
- **Cost optimization:** job clusters + auto-termination, spot/low-priority VMs, right-sizing, Photon, minimize shuffles, Parquet/Delta, **serverless** for spiky/ad-hoc, lifecycle tiering of cold data, avoid `SELECT *`.

### Interview Q&A

**Q: How do you handle failures in ADF pipelines?**
Configure **retries + timeouts** on activities, use **on-failure dependency paths** to run cleanup/alert activities, send notifications via **Logic Apps/Azure Monitor alerts**, and design pipelines to be **idempotent/restartable** so reruns don't corrupt data.

**Q: How do you monitor pipelines in production?**
Use the **ADF Monitor** view and **Azure Monitor/Log Analytics** for run history, metrics, and **alerts**; for Databricks, use **job run logs and the Spark UI**. Set proactive alerts on failures and SLAs.

**Q: How do you optimize cost on Azure data platforms?**
Use **job clusters with auto-termination**, **spot/low-priority** VMs, right-size compute, enable **Photon/AQE**, store in **Parquet/Delta**, use **serverless** for ad-hoc, tier/archive cold data, and minimize shuffles and full scans.

---

## 16. Scenario / Design Questions

Highly likely for experienced candidates — practice answering with a structure.

1. **Design a pipeline** to ingest daily vendor files into ADLS, transform via medallion in Databricks, serve to Power BI. → ADF event/schedule trigger → Copy to Bronze → Databricks notebooks for Silver/Gold (MERGE) → serve via SQL endpoint/Fabric → Power BI (Direct Lake). Add retries, watermark, alerts.
2. **Incremental load** of a huge source table nightly. → Watermark pattern (Lookup → filtered Copy → update watermark) or CDC.
3. **Large fact × small dimension join is slow** in Spark. → Broadcast the dimension, partition/bucket fact on join key, cache dim, OPTIMIZE/Z-Order, AQE.
4. **Files arrive at random times.** → Event-based trigger (storage events) or Auto Loader for continuous incremental ingestion.
5. **Schema drift** in incoming JSON. → Auto Loader schema evolution / `mergeSchema`, quarantine bad records, alert.
6. **Nightly job failed mid-run.** → Atomic Delta commits + idempotent MERGE + checkpoints + retry → safe rerun, no partial corruption.
7. **Migrate from on-prem SQL to Azure.** → SHIR to connect on-prem → Copy to ADLS (staged Parquet) → transform → load Synapse/Lakehouse; validate with reconciliation counts.
8. **Reduce cost of an expensive pipeline.** → Job clusters + auto-terminate, spot VMs, fewer shuffles, serverless SQL for ad-hoc, lifecycle tiering.
9. **Implement SCD Type 2** for a customer dimension. → Delta `MERGE` with `is_current`/`start_date`/`end_date`.
10. **One pipeline, many tables** (metadata-driven). → Control table + Lookup + ForEach with parameterized datasets.

**Answer framework:** Requirements/SLA → Ingestion (batch/stream, incremental) → Storage zones (medallion) → Transform (Spark/Delta) → Orchestration (ADF triggers/dependencies) → Quality & monitoring → Serving (warehouse/Power BI) → Security & cost.

---

## 17. Common Code / Config Snippets

### 1. Read source → write Bronze Delta (PySpark)

```python
df = spark.read.option("header", True).csv("/mnt/landing/sales.csv")
df.write.format("delta").mode("append").saveAsTable("bronze.sales")
```

### 2. MERGE / Upsert (SCD Type 1)

```sql
MERGE INTO silver.customers t USING staging.customers s ON t.id = s.id
WHEN MATCHED THEN UPDATE SET *
WHEN NOT MATCHED THEN INSERT *;
```

### 3. Deduplicate keeping latest

```python
from pyspark.sql.window import Window
from pyspark.sql import functions as F
w = Window.partitionBy("id").orderBy(F.col("updated_at").desc())
(df.withColumn("rn", F.row_number().over(w)).filter("rn=1").drop("rn")
   .write.format("delta").mode("overwrite").saveAsTable("silver.customers"))
```

### 4. Incremental file ingest (Auto Loader)

```python
(spark.readStream.format("cloudFiles")
   .option("cloudFiles.format", "json")
   .option("cloudFiles.schemaLocation", "/chk/schema")
   .load("/mnt/landing")
 .writeStream.option("checkpointLocation", "/chk/cp")
   .trigger(availableNow=True).toTable("bronze.events"))
```

### 5. ADF watermark source query (parameterized)

```sql
SELECT * FROM orders
WHERE LastModifiedDate > '@{activity('LookupWatermark').output.firstRow.wm}';
```

### 6. Serverless SQL over the lake (Synapse)

```sql
SELECT TOP 100 *
FROM OPENROWSET(
  BULK 'https://acct.dfs.core.windows.net/gold/sales/',
  FORMAT = 'PARQUET') AS rows;
```

### 7. Time travel / rollback

```sql
SELECT * FROM sales VERSION AS OF 10;
RESTORE TABLE sales TO VERSION AS OF 10;
```

### 8. Optimize + Vacuum

```sql
OPTIMIZE sales ZORDER BY (customer_id);
VACUUM sales RETAIN 168 HOURS;
```

---

## 18. Tricky / Gotcha Questions

**1. Linked Service ≠ Dataset.** Linked Service = connection; Dataset = the specific data within it. Mixing these up is a classic stumble.

**2. ADF Copy Activity does NOT do complex transforms** — use Data Flows or Databricks for that.

**3. Self-hosted IR is required for on-prem** — Azure IR can't reach private networks.

**4. Tumbling window vs schedule trigger:** only tumbling windows support **backfill, dependencies, and per-window retry**.

**5. ForEach runs in parallel by default** — set sequential when order matters (e.g., dependent loads).

**6. `DROP` on a managed table deletes data; external table keeps files.**

**7. VACUUM below 7 days breaks time travel** (and Databricks blocks it without an override flag).

**8. Delta = Parquet + log** — reading the raw Parquet directly bypasses the log and can give wrong/stale results.

**9. Fabric is SaaS, ADF is PaaS** — same DI heritage, different deployment/billing model.

**10. OneLake stores ONE copy** in Delta — Lakehouse and Warehouse both sit on it; **shortcuts** avoid duplication.

**11. Incremental loads need idempotency** — without MERGE/partition-overwrite, reruns duplicate data.

**12. Hash vs Round-Robin distribution** in Synapse — wrong choice causes massive data movement (shuffle) on joins.

---

## 19. Rapid-Fire One-Liners

- **ADF** = serverless cloud ETL/ELT **orchestration** (PaaS).
- **Linked Service** = connection; **Dataset** = data pointer; **Activity** = step; **Pipeline** = workflow; **IR** = compute.
- **IR types:** Azure (cloud), Self-hosted (on-prem/private), Azure-SSIS (legacy SSIS).
- **Triggers:** schedule, **tumbling window** (backfill/deps/retry), event-based.
- **Incremental load** = watermark pattern (Lookup → filtered Copy → update watermark).
- **Copy** = move; **Data Flow** = visual transforms; **Databricks** = code-first transforms.
- **ADLS Gen2** = Blob + hierarchical namespace + ACLs.
- **ELT** > ETL in cloud; transform in the lake/warehouse.
- **Delta** = Parquet + `_delta_log`; ACID, time travel, schema evolution, MERGE.
- **OPTIMIZE** compacts; **ZORDER** data skipping; **VACUUM** cleans; **Liquid Clustering** modern.
- **Medallion:** Bronze (raw) → Silver (clean) → Gold (curated).
- **Fabric** = SaaS unified analytics on **OneLake** (one Delta copy); **Direct Lake** for Power BI.
- **Lakehouse** (Spark) vs **Warehouse** (T-SQL) — both on OneLake.
- **Synapse:** dedicated SQL (MPP), serverless SQL (lake on-demand), Spark, pipelines.
- **Distributions:** Hash (big facts), Round-Robin (staging), Replicated (small dims).
- **Star schema** = fact + denormalized dims; **SCD2** keeps history via MERGE.
- **Secrets** in **Key Vault**; **managed identity** = no-credential auth; **Entra ID** for identity.
- **CI/CD:** Git + ARM templates (ADF), Repos + Asset Bundles (Databricks).
- **Purview** = enterprise governance/lineage/catalog.
- **Cost:** job clusters + auto-terminate, spot VMs, serverless, Parquet/Delta, fewer shuffles.

---

## 20. Last-Minute Checklist

The hour before:

- [ ] The Azure data flow: Sources → ADF → ADLS → Databricks/Synapse → Synapse/Fabric → Power BI.
- [ ] ETL vs ELT; PaaS (ADF/Synapse) vs SaaS (Fabric).
- [ ] ADF building blocks: Linked Service, Dataset, Activity, Pipeline, IR.
- [ ] IR types and when to use Self-hosted.
- [ ] Triggers: schedule vs tumbling window vs event; backfill.
- [ ] **Incremental load / watermark pattern** (be ready to whiteboard it).
- [ ] Copy vs Data Flow vs Databricks; ForEach/Lookup metadata-driven pipelines.
- [ ] Delta: ACID, time travel, schema evolution, OPTIMIZE/ZORDER/VACUUM, MERGE.
- [ ] Medallion layers and what each does.
- [ ] **Microsoft Fabric:** OneLake, Lakehouse vs Warehouse, Direct Lake, shortcuts, ADF vs Fabric DF.
- [ ] Synapse: dedicated vs serverless SQL, distributions, COPY INTO.
- [ ] Modeling: star vs snowflake, fact/dim, SCD1 vs SCD2.
- [ ] Security: Key Vault, managed identity, Entra ID, Purview, RBAC.
- [ ] CI/CD for ADF (Git + ARM) and Databricks (Repos/DABs).
- [ ] Error handling, monitoring, cost optimization.
- [ ] Practice 3–4 **scenario designs** out loud using the framework in §16.

**Interview tips:** Lead with the **ELT + medallion + Delta** story for design questions. For ADF, nail the **building blocks + incremental watermark pattern**. For Fabric, emphasize **OneLake + one copy + Direct Lake + SaaS**. Always mention **idempotency, monitoring, security, and cost** — that's what separates a senior data engineer from a junior one.

---

*Good luck — read it top-to-bottom once, then drill the ADF incremental pattern (§7), Fabric/OneLake (§10), Synapse distributions (§11), and the scenario designs (§16). Those decide Azure DE interviews.*
