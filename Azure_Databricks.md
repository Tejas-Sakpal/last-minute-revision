# Azure Databricks Interview — Last-Minute Prep Guide

> **How to use this:** Skim the **Theory** for crisp definitions, read the **Code/Config** to see it in practice, and rehearse the **Interview Q&A** out loud. Bold lines are the one-liners interviewers want to hear. Built for a 2–4 hour final revision.
>
> *Targets Azure Databricks on Spark 3.x with Delta Lake and Unity Catalog. Code is PySpark / SQL / Python as noted.*

---

## Table of Contents

1. [What is Azure Databricks & Architecture](#1-what-is-azure-databricks--architecture)
2. [Control Plane vs Data Plane](#2-control-plane-vs-data-plane)
3. [Clusters: Types, Modes & Autoscaling](#3-clusters-types-modes--autoscaling)
4. [Workspace, Notebooks & DBFS](#4-workspace-notebooks--dbfs)
5. [Delta Lake Fundamentals](#5-delta-lake-fundamentals)
6. [Delta Lake Features: ACID, Time Travel, Schema Evolution](#6-delta-lake-features)
7. [Managed vs External Tables](#7-managed-vs-external-tables)
8. [Medallion Architecture (Bronze/Silver/Gold)](#8-medallion-architecture)
9. [Unity Catalog & Governance](#9-unity-catalog--governance)
10. [Delta Live Tables (DLT)](#10-delta-live-tables-dlt)
11. [Jobs, Workflows & Scheduling](#11-jobs-workflows--scheduling)
12. [Autoloader & Structured Streaming](#12-autoloader--structured-streaming)
13. [Performance & Optimization](#13-performance--optimization)
14. [Photon, Security & Secrets](#14-photon-security--secrets)
15. [Azure Integration (ADLS, ADF, Synapse, Key Vault)](#15-azure-integration)
16. [Common Coding / Scenario Tasks](#16-common-coding--scenario-tasks)
17. [Tricky / Gotcha Questions](#17-tricky--gotcha-questions)
18. [Rapid-Fire One-Liners](#18-rapid-fire-one-liners)
19. [Last-Minute Checklist](#19-last-minute-checklist)

---

## 1. What is Azure Databricks & Architecture

### Theory

**Azure Databricks** is a **first-party Microsoft Azure service** (jointly developed by Microsoft and Databricks) that provides a **unified analytics platform** built on **Apache Spark**. It combines data engineering, data science, ML, and BI on a **lakehouse** architecture.

**Lakehouse** = combines the low-cost, open storage of a **data lake** with the **ACID transactions, schema, and performance** of a **data warehouse** — eliminating the need for separate systems. Powered by **Delta Lake**.

**Why Databricks over plain Spark/HDInsight:** managed/optimized Spark runtime, collaborative notebooks, auto-scaling clusters, Delta Lake, Unity Catalog governance, MLflow, Photon engine, and tight Azure integration (AAD, ADLS, ADF).

**Key components:** Workspace, Clusters, Notebooks, Jobs/Workflows, Delta Lake, Unity Catalog, Databricks SQL, MLflow, Delta Live Tables.

### Code

```python
# A Databricks notebook cell runs against an attached cluster
df = spark.read.format("delta").load("/mnt/data/sales")
display(df)               # Databricks-native rich table/visualization
```

### Interview Q&A

**Q: What is Azure Databricks?**
A **managed, Azure-native** Apache Spark–based analytics platform offering a **lakehouse** — unified data engineering, analytics, and ML with collaborative notebooks, optimized Spark, Delta Lake, and deep Azure integration.

**Q: What is a lakehouse?**
An architecture that merges a **data lake's** cheap, open, scalable storage with a **data warehouse's** ACID transactions, schema enforcement, and performance — implemented via **Delta Lake** on cloud object storage.

**Q: How is Azure Databricks different from open-source Spark?**
It adds a managed, performance-tuned runtime (Photon), Delta Lake, Unity Catalog governance, auto-scaling/auto-terminating clusters, collaborative notebooks, MLflow, job orchestration, and native Azure security (Azure AD/Entra ID, ADLS).

**Q: How is it billed?**
By **DBU (Databricks Unit)** — a unit of processing capability per hour — **plus** the underlying Azure VM/storage costs. Cluster type and size determine DBU consumption.

---

## 2. Control Plane vs Data Plane

### Theory

Databricks splits responsibilities into two planes:

- **Control Plane (Databricks-managed, in Databricks' Azure subscription):** the backend services — web UI, notebooks, job scheduler, cluster manager, REST APIs. Stores **metadata and notebook commands** (encrypted). It does **not** hold your data.
- **Data Plane / Compute Plane (in YOUR Azure subscription):** where clusters actually run and **your data is processed**. Data lives in **your ADLS/storage**. This keeps data within your network/tenant for security and compliance.

**Classic** compute plane runs in your VNet; **Serverless** compute plane runs in Databricks' account (managed, faster startup).

### Interview Q&A

**Q: Explain the control plane vs data plane.**
The **control plane** (Databricks' subscription) hosts management services — UI, job scheduling, cluster orchestration, and metadata. The **data plane** (your Azure subscription) is where clusters spin up and **your data is actually processed and stored**, so data stays in your tenant.

**Q: Where does your data physically reside?**
In your own Azure storage (ADLS Gen2) within the **data plane** — never in the control plane. The control plane only stores metadata and command results.

**Q: Classic vs serverless compute?**
**Classic** clusters run in **your** VNet/subscription (full network control). **Serverless** runs in Databricks-managed infrastructure with near-instant startup and no cluster management, at the cost of less network customization.

---

## 3. Clusters: Types, Modes & Autoscaling

### Theory

A **cluster** is a set of VMs (one **driver** + **workers**) running the Databricks Runtime.

**Cluster types:**

- **All-Purpose (Interactive) cluster** — for interactive development in notebooks, shared by multiple users; can be restarted; **higher DBU cost**.
- **Job cluster** — created **automatically** for a scheduled job and **terminated when it finishes**; cheaper, isolated, reproducible. **Preferred for production**.
- **SQL Warehouse** — compute for Databricks SQL / BI queries (Serverless, Pro, Classic).

**Access modes (Unity Catalog):** Single user, Shared, No isolation shared.

**Autoscaling** — Databricks automatically adds/removes workers between a min and max based on load, saving cost. **Auto-termination** shuts down idle clusters after N minutes.

**Databricks Runtime (DBR)** — preconfigured image (Spark + Delta + libraries). Variants: Standard, **ML** (adds ML libs/GPU), **Photon** (vectorized engine).

### Code

```json
// Job cluster spec (JSON) - ephemeral, cost-efficient
{
  "num_workers": 2,
  "spark_version": "14.3.x-scala2.12",
  "node_type_id": "Standard_DS3_v2",
  "autoscale": { "min_workers": 2, "max_workers": 8 },
  "autotermination_minutes": 30
}
```

### Interview Q&A

**Q: All-purpose vs job cluster?**
An **all-purpose** cluster is for **interactive** notebook development, can be shared and manually restarted, and costs more (DBU). A **job cluster** is spun up **automatically for a single job** and **terminated on completion** — cheaper, isolated, and the recommended choice for production pipelines.

**Q: How does autoscaling work?**
Databricks monitors pending tasks and **adds workers** when load rises (up to `max_workers`) and **removes** them when idle (down to `min_workers`), balancing performance and cost without manual intervention.

**Q: What is auto-termination and why use it?**
Automatically shutting down a cluster after a set idle period to **avoid paying for idle compute** — important for cost control on interactive clusters.

**Q: What is the Databricks Runtime?**
A managed, optimized image bundling a tuned Spark, Delta Lake, and common libraries. Specialized versions: **ML** (ML/DL libraries, GPU support) and **Photon** (native vectorized query engine for speed).

**Q: Driver vs worker node?**
The **driver** hosts the SparkSession, builds the DAG, and coordinates; **workers** (executors) run the tasks and hold data partitions. A cluster always has one driver and zero-or-more workers.

---

## 4. Workspace, Notebooks & DBFS

### Theory

- **Workspace** — the environment containing notebooks, folders, libraries, repos, dashboards, and configs.
- **Notebook** — interactive document with cells; supports **multiple languages** via magic commands (`%python`, `%sql`, `%scala`, `%r`, `%md`, `%sh`, `%fs`).
- **DBFS (Databricks File System)** — an abstraction layer over cloud object storage, mounted at `/dbfs`. (Note: with Unity Catalog, **Volumes** are now preferred over DBFS mounts for governed file access.)
- **`dbutils`** — utility library for filesystem (`dbutils.fs`), secrets (`dbutils.secrets`), widgets (`dbutils.widgets`), and notebook workflow (`dbutils.notebook.run`).
- **Repos** — Git integration for version-controlled notebooks/code.

### Code

```python
# Multi-language with magics
# %sql
#   SELECT * FROM sales LIMIT 10

# dbutils file ops
dbutils.fs.ls("/mnt/data")
dbutils.fs.mkdirs("/mnt/data/new")

# Parameters via widgets
dbutils.widgets.text("run_date", "2025-01-01")
run_date = dbutils.widgets.get("run_date")

# Call another notebook (modular pipelines)
dbutils.notebook.run("./child_notebook", timeout_seconds=600, arguments={"date": run_date})
```

### Interview Q&A

**Q: Can a notebook use multiple languages?**
Yes — via **magic commands** (`%python`, `%sql`, `%scala`, `%r`). The default language is set per notebook; magics override per cell. `%md` for markdown, `%sh` for shell, `%fs` for DBFS.

**Q: What is `dbutils`?**
A built-in utility module for filesystem operations (`dbutils.fs`), secret retrieval (`dbutils.secrets`), parameter widgets (`dbutils.widgets`), and chaining notebooks (`dbutils.notebook.run`).

**Q: What is DBFS, and what replaces it under Unity Catalog?**
**DBFS** is a filesystem abstraction over cloud storage. Under Unity Catalog, **Volumes** provide governed, access-controlled file storage and are preferred over legacy DBFS mounts.

**Q: How do you pass parameters into a notebook?**
With **widgets** (`dbutils.widgets.text/get`), which also let jobs/ADF pass arguments at runtime.

---

## 5. Delta Lake Fundamentals

### Theory

**Delta Lake** is the open-source storage layer that brings **reliability** to data lakes. A Delta table = **Parquet data files** + a **transaction log** (`_delta_log`, JSON + checkpoints) that records every change.

**The transaction log (`_delta_log`) is the heart of Delta** — it provides ACID, time travel, and concurrency by recording an ordered series of atomic commits.

**Benefits over plain Parquet:** ACID transactions, schema enforcement & evolution, time travel (versioning), upserts/deletes (MERGE), scalable metadata, unified batch + streaming.

### Code

```python
# Write a Delta table
df.write.format("delta").mode("overwrite").saveAsTable("sales")

# Read
spark.read.format("delta").table("sales")

# Upsert with MERGE (SCD-style)
spark.sql("""
MERGE INTO sales t
USING updates s
ON t.id = s.id
WHEN MATCHED THEN UPDATE SET *
WHEN NOT MATCHED THEN INSERT *
""")
```

### Interview Q&A

**Q: What is Delta Lake and how does it differ from Parquet?**
Delta Lake is a storage layer over **Parquet** plus a **transaction log** (`_delta_log`). Unlike plain Parquet, it adds **ACID transactions, schema enforcement/evolution, time travel, MERGE/UPDATE/DELETE, and unified batch+streaming**.

**Q: What is the `_delta_log`?**
The **transaction log** — an ordered record of every commit (as JSON files, periodically compacted into Parquet checkpoints). It's what enables ACID guarantees, time travel, and concurrency control.

**Q: How does Delta handle concurrent reads/writes?**
Through **optimistic concurrency control**: each writer reads the current table version, stages changes, and commits atomically; if a conflicting commit happened, it retries. Readers always see a **consistent snapshot** (snapshot isolation).

**Q: Does Delta support DELETE/UPDATE?**
Yes — unlike raw Parquet, Delta supports `DELETE`, `UPDATE`, and `MERGE`, rewriting affected files and logging the change.

---

## 6. Delta Lake Features

### Theory

**ACID transactions** — Atomicity, Consistency, Isolation, Durability on the data lake via the transaction log.

**Time Travel** — query previous versions of a table by **version number** or **timestamp** (great for audits, rollback, reproducibility). Old files retained per **retention period** (default 7 days for `VACUUM`).

**Schema enforcement** — rejects writes that don't match the table schema (prevents corruption).

**Schema evolution** — allow new columns to be added automatically with `mergeSchema` / `autoMerge`.

**OPTIMIZE & Z-ORDER** — compacts small files (`OPTIMIZE`) and co-locates related data by columns (`ZORDER BY`) for faster filtering. **Liquid Clustering** is the newer alternative to partitioning + Z-Order.

**VACUUM** — removes data files no longer referenced and older than the retention threshold (cleans up storage; limits time travel afterward).

### Code

```sql
-- Time travel
SELECT * FROM sales VERSION AS OF 5;
SELECT * FROM sales TIMESTAMP AS OF '2025-01-01';

-- Restore to a prior version
RESTORE TABLE sales TO VERSION AS OF 5;

-- Schema evolution on write (PySpark)
-- df.write.option("mergeSchema", "true").mode("append").saveAsTable("sales")

-- Optimize + Z-Order + cleanup
OPTIMIZE sales ZORDER BY (customer_id);
VACUUM sales RETAIN 168 HOURS;     -- 7 days
DESCRIBE HISTORY sales;            -- see all versions/operations
```

### Interview Q&A

**Q: What is time travel and how do you use it?**
The ability to query or restore a **previous version** of a Delta table using `VERSION AS OF` / `TIMESTAMP AS OF`. Useful for **audits, debugging, rollback, and reproducible ML**. View versions with `DESCRIBE HISTORY`.

**Q: Schema enforcement vs schema evolution?**
**Enforcement** rejects writes that don't match the table's schema (protects integrity). **Evolution** lets the schema **change intentionally** (e.g., add columns) using `mergeSchema`/`autoMerge`.

**Q: What do OPTIMIZE and ZORDER do?**
`OPTIMIZE` **compacts many small files** into fewer large ones (fixing the small-file problem). `ZORDER BY` **co-locates** rows with similar values in the same files, so queries filtering on those columns read far less data (better data skipping).

**Q: What is VACUUM and its risk?**
`VACUUM` deletes unreferenced data files older than the retention period to reclaim storage. **Risk:** vacuuming below the default 7-day retention **breaks time travel** to older versions and can corrupt long-running readers.

**Q: What is Liquid Clustering?**
A newer Delta feature that **replaces static partitioning and Z-Order** with automatic, flexible data clustering on chosen keys — avoids over/under-partitioning and adapts as data grows.

---

## 7. Managed vs External Tables

### Theory

- **Managed table** — Databricks manages **both metadata and the underlying data files** (stored in the metastore's/catalog's managed location). `DROP TABLE` **deletes the data** too.
- **External (Unmanaged) table** — you specify a `LOCATION`; Databricks manages **only the metadata**. `DROP TABLE` removes the metadata but **leaves the data files** in storage.

**Use external** when data is shared with other tools/systems or you want to retain files independently; **use managed** for full lifecycle control within Databricks.

### Code

```sql
-- Managed: data lives in the catalog's managed storage
CREATE TABLE managed_sales (id INT, amt DOUBLE);

-- External: you control the location
CREATE TABLE ext_sales (id INT, amt DOUBLE)
USING DELTA
LOCATION 'abfss://data@acct.dfs.core.windows.net/sales';

DROP TABLE managed_sales;   -- deletes data + metadata
DROP TABLE ext_sales;       -- deletes metadata only; files remain
```

### Interview Q&A

**Q: Managed vs external table — key difference?**
For a **managed** table Databricks controls both metadata and data files, so `DROP` deletes the data. For an **external** table you provide a `LOCATION`; Databricks manages only metadata, so `DROP` leaves the files intact.

**Q: When would you use an external table?**
When the data is **shared across systems**, must persist independently of the table definition, or already exists in a specific storage location you want to keep.

---

## 8. Medallion Architecture

### Theory

A layered data-design pattern using Delta tables to progressively refine data quality:

- **Bronze (Raw):** ingests source data **as-is** (append-only, full history, minimal transformation). Preserves exactly what arrived — auditable and reprocessable.
- **Silver (Cleansed/Conformed):** **cleaned, deduplicated, schema-enforced, joined/validated** data. The "single source of truth" enterprise view.
- **Gold (Curated/Aggregated):** **business-level aggregates**, feature tables, and marts optimized for **BI/reporting and ML**.

**Why it matters:** data-quality issues become **traceable and re-processable**; each layer has a clear responsibility; you can rebuild downstream layers from Bronze.

### Code

```python
# Bronze - raw ingest
raw = spark.read.json("/mnt/landing/events")
raw.write.format("delta").mode("append").saveAsTable("bronze.events")

# Silver - clean & dedupe
from pyspark.sql import functions as F
(spark.table("bronze.events")
   .dropDuplicates(["event_id"])
   .filter(F.col("user_id").isNotNull())
   .write.format("delta").mode("overwrite").saveAsTable("silver.events"))

# Gold - business aggregate
(spark.table("silver.events")
   .groupBy("country").agg(F.count("*").alias("event_count"))
   .write.format("delta").mode("overwrite").saveAsTable("gold.events_by_country"))
```

### Interview Q&A

**Q: Explain the medallion architecture.**
A three-layer pattern: **Bronze** stores raw source data as-is; **Silver** holds cleaned, deduplicated, schema-validated data; **Gold** provides curated, aggregated tables for BI and ML. It improves data quality, traceability, and reprocessability.

**Q: What transformations happen at Silver?**
**Cleaning, deduplication, schema enforcement, type casting, null handling, and joins/conforming** — turning raw Bronze into a reliable, query-ready enterprise dataset.

**Q: Why keep a Bronze layer if it's raw?**
For **auditability and reprocessing** — if business logic changes or a bug is found, you can rebuild Silver/Gold from the untouched raw history without re-ingesting from source.

---

## 9. Unity Catalog & Governance

### Theory

**Unity Catalog (UC)** is Databricks' **centralized governance layer** for all data and AI assets across workspaces. It **replaces the legacy Hive metastore**.

**Three-level namespace:** `catalog.schema.table` (vs the old two-level `schema.table`).

**Key capabilities:**

- **Fine-grained access control** — down to **catalog, schema, table, row, and column** level (with row filters & column masks), using standard `GRANT`/`REVOKE`.
- **Centralized metastore** — one governance model shared across multiple workspaces.
- **Automated data lineage** — table- and column-level lineage tracking.
- **Unified audit logs** for compliance.
- **Identity** via Azure Entra ID (AAD) groups/users.
- **Delta Sharing** — open protocol to share data across orgs/platforms securely.
- **Volumes** — governed storage for non-tabular files.

**Hierarchy:** Metastore → Catalog → Schema (database) → Table/View/Volume/Function.

### Code

```sql
-- Three-level namespace
SELECT * FROM main.sales.transactions;

-- Grants (fine-grained)
GRANT SELECT ON TABLE main.sales.transactions TO `analysts`;
GRANT USAGE ON SCHEMA main.sales TO `analysts`;

-- Row filter / column mask (governance)
CREATE FUNCTION main.sales.region_filter(region STRING)
RETURN region = current_user_region();
ALTER TABLE main.sales.transactions
SET ROW FILTER main.sales.region_filter ON (region);
```

### Interview Q&A

**Q: What is Unity Catalog and why use it?**
A **centralized governance** solution for data and AI across all workspaces. It replaces the Hive metastore and provides **fine-grained access control (down to row/column), automated lineage, unified auditing, and cross-workspace/org data sharing** under one model.

**Q: What's the three-level namespace?**
`catalog.schema.table` — Unity Catalog adds the **catalog** level above the traditional `schema.table`, enabling logical isolation (e.g., by environment or business unit).

**Q: Unity Catalog vs Hive metastore?**
Hive metastore is **per-workspace** with coarse-grained, two-level naming and no native lineage. **Unity Catalog** is **account-level**, shared across workspaces, with three-level naming, row/column-level security, lineage, and audit logs.

**Q: What is Delta Sharing?**
An **open protocol** to securely share live Delta data with other organizations or tools (Power BI, pandas, other clouds) **without copying** it.

**Q: How is identity managed?**
Through **Azure Entra ID (Azure AD)** — users and groups are synced and used for grants, so governance aligns with corporate identity.

---

## 10. Delta Live Tables (DLT)

### Theory

**Delta Live Tables (DLT)** is a **declarative framework** for building reliable ETL pipelines. You declare the **transformations and data-quality rules**, and Databricks manages **orchestration, dependencies, error handling, retries, autoscaling, and monitoring**.

**Key features:**

- **Declarative** (`@dlt.table`) — define *what* the table is, not the procedural steps.
- **Automatic dependency resolution** — DLT builds the DAG between tables.
- **Expectations** — data-quality constraints (`@dlt.expect`) that warn, drop, or fail on bad rows.
- **Streaming + batch** in one framework; supports incremental processing.
- **Auto-managed infrastructure** and lineage.

**DLT vs notebooks:** notebooks are **imperative and manually orchestrated**; DLT is **declarative** with built-in quality, dependency management, and operational tooling.

### Code

```python
import dlt
from pyspark.sql import functions as F

@dlt.table(comment="Cleaned silver events")
@dlt.expect_or_drop("valid_id", "event_id IS NOT NULL")   # quality rule
def silver_events():
    return (dlt.read_stream("bronze_events")
              .dropDuplicates(["event_id"]))

@dlt.table
def gold_by_country():
    return dlt.read("silver_events").groupBy("country").count()
```

### Interview Q&A

**Q: What is Delta Live Tables?**
A **declarative ETL framework** where you define table transformations and data-quality expectations, and Databricks **automatically handles orchestration, dependencies, retries, scaling, and monitoring** — producing reliable, observable pipelines.

**Q: DLT vs regular notebooks?**
Notebooks are **imperative** — you write and orchestrate each step manually. DLT is **declarative**: you describe the target tables and quality rules, and the framework manages the **DAG, incremental processing, error handling, and lineage** for you.

**Q: What are expectations in DLT?**
**Data-quality constraints** (`@dlt.expect`, `expect_or_drop`, `expect_or_fail`) that validate rows — they can **track/warn**, **drop** bad rows, or **fail** the pipeline, giving built-in quality enforcement.

---

## 11. Jobs, Workflows & Scheduling

### Theory

**Databricks Workflows / Jobs** orchestrate tasks (notebooks, Python scripts, JARs, SQL, DLT pipelines) with **dependencies, scheduling, retries, and alerts**.

- A **Job** can have **multiple tasks** in a DAG (task dependencies).
- Triggers: **scheduled (cron)**, **continuous**, **file arrival**, or **manual/API**.
- Run jobs on **job clusters** (ephemeral, recommended) for cost/isolation.
- Parameters passed via widgets; supports **retries**, **timeouts**, **email/Teams alerts**, and **task values** to pass data between tasks.

### Code

```python
# Pass values between tasks
dbutils.jobs.taskValues.set(key="row_count", value=12345)
# Downstream task:
n = dbutils.jobs.taskValues.get(taskKey="ingest", key="row_count")
```

### Interview Q&A

**Q: How do you orchestrate a multi-step pipeline in Databricks?**
Use **Workflows (Jobs)** with multiple **tasks** arranged in a DAG of dependencies, each running a notebook/script/DLT pipeline. Configure **schedules (cron), retries, timeouts, alerts**, and run on **job clusters** for cost efficiency.

**Q: Why run production jobs on job clusters?**
They're **created per run and terminated on completion** — cheaper, isolated, and reproducible, avoiding the cost and state of long-running all-purpose clusters.

**Q: How do you schedule a job?**
Via a **cron expression** in the job's schedule (or continuous/file-arrival triggers), or externally through the **Jobs REST API** / Azure Data Factory.

---

## 12. Autoloader & Structured Streaming

### Theory

**Auto Loader** (`cloudFiles`) **incrementally and efficiently ingests new files** as they land in cloud storage, without re-listing everything. It tracks processed files via a **scalable checkpoint** (RocksDB) and supports **schema inference and evolution**.

**Structured Streaming** processes data incrementally as micro-batches (or continuously) with **exactly-once** guarantees via **checkpoints** and Delta. `trigger(availableNow=True)` enables incremental **batch** processing.

### Code

```python
# Auto Loader: incremental ingestion into Bronze
(spark.readStream
   .format("cloudFiles")
   .option("cloudFiles.format", "json")
   .option("cloudFiles.schemaLocation", "/mnt/chk/schema")
   .load("/mnt/landing/events")
 .writeStream
   .option("checkpointLocation", "/mnt/chk/events")
   .trigger(availableNow=True)            # process all new files then stop
   .toTable("bronze.events"))
```

### Interview Q&A

**Q: What is Auto Loader and why use it over a plain read?**
`cloudFiles` Auto Loader **incrementally ingests only new files** from cloud storage using an efficient checkpoint/notification mechanism — avoiding costly full directory listings, and supporting **schema inference and evolution** at scale.

**Q: How does streaming guarantee exactly-once?**
Through **checkpointing** (offsets/state) combined with Delta's atomic commits — on failure it resumes from the last checkpoint without duplicating or losing data.

**Q: Auto Loader vs COPY INTO?**
**Auto Loader** is for **continuous/incremental** ingestion of large or ongoing file streams. **COPY INTO** is a **SQL command for idempotent batch** loads of files — simpler for smaller, periodic loads.

---

## 13. Performance & Optimization

### Theory

Be ready to list concrete levers:

1. **File compaction** — `OPTIMIZE` to fix the small-file problem.
2. **Data skipping & Z-Order / Liquid Clustering** — co-locate filter columns; Delta auto-collects min/max stats per file.
3. **Partitioning** — partition large tables by low-cardinality columns (e.g., date); avoid over-partitioning.
4. **Caching** — `cache()`/Delta cache (SSD) for reused data.
5. **Broadcast joins** for large × small.
6. **Photon** — vectorized engine for big speedups on SQL/DataFrame ops.
7. **AQE (Adaptive Query Execution)** — runtime re-optimization (skew handling, partition coalescing).
8. **Right cluster sizing & autoscaling**; use **job clusters**.
9. **Minimize shuffles**; filter/select early.
10. **Use Delta + Parquet** (columnar), not CSV/JSON for analytics.

### Code

```sql
OPTIMIZE events ZORDER BY (user_id);          -- compaction + clustering
ANALYZE TABLE events COMPUTE STATISTICS;      -- better plans
SET spark.databricks.delta.optimizeWrite.enabled = true;   -- auto small-file handling
SET spark.databricks.delta.autoCompact.enabled = true;
```

### Interview Q&A

**Q: How do you tune a slow Databricks/Delta pipeline?**
Compact small files with **OPTIMIZE**, apply **Z-Order/Liquid Clustering** and partitioning for **data skipping**, **broadcast** small join tables, **cache** reused data, enable **Photon** and **AQE**, minimize shuffles, size the cluster appropriately, and use **Delta/Parquet** formats.

**Q: What's the "small file problem" and the fix?**
Many tiny files (from frequent streaming/appends) cause excessive metadata and slow reads. Fix with **`OPTIMIZE`** (compaction), **Optimized Writes**, and **Auto Compaction**.

**Q: What is data skipping?**
Delta stores **min/max statistics per file** in the transaction log; queries with filters **skip files** that can't contain matching rows — amplified by **Z-Order** clustering on the filter columns.

**Q: What is AQE?**
**Adaptive Query Execution** re-optimizes the plan at **runtime** using actual statistics — coalescing shuffle partitions, switching join strategies, and handling skew automatically.

---

## 14. Photon, Security & Secrets

### Theory

- **Photon** — Databricks' **native, vectorized C++ query engine** that accelerates SQL and DataFrame workloads (no code change), improving price/performance for analytical queries.
- **Secrets** — never hardcode credentials. Store in a **secret scope** (Databricks-backed or **Azure Key Vault–backed**) and retrieve with `dbutils.secrets.get`.
- **Security:** Azure Entra ID (AAD) auth, RBAC, Unity Catalog grants, network isolation (VNet injection, Private Link), encryption at rest/in transit, credential passthrough.

### Code

```python
# Retrieve a secret (never hardcode)
key = dbutils.secrets.get(scope="kv-scope", key="storage-key")

# Use it to configure storage access
spark.conf.set("fs.azure.account.key.acct.dfs.core.windows.net", key)
```

### Interview Q&A

**Q: What is Photon?**
A **vectorized native execution engine** (C++) that transparently speeds up SQL/DataFrame workloads with better CPU efficiency — enabled at the cluster level, no code changes needed.

**Q: How do you handle secrets/credentials securely?**
Store them in a **secret scope** (ideally **Azure Key Vault–backed**) and fetch at runtime via `dbutils.secrets.get` — never hardcode in notebooks. Values are **redacted** in output.

**Q: How is Azure Databricks secured?**
**Azure Entra ID** authentication and SSO, **Unity Catalog** fine-grained authorization, **RBAC**, network controls (VNet injection, **Private Link**, NSGs), and **encryption** at rest and in transit.

---

## 15. Azure Integration

### Theory

- **ADLS Gen2** — primary data lake storage; access via **service principal + OAuth**, **managed identity**, or **Unity Catalog external locations/credentials** (preferred). Legacy: mounts with account keys/SAS.
- **Azure Data Factory (ADF)** — orchestration; runs Databricks notebooks/jobs as pipeline activities (common ingestion → transform pattern).
- **Azure Key Vault** — backs secret scopes for credentials.
- **Azure Synapse / SQL DW** — serve curated Gold data to the warehouse/BI; use the Synapse connector or Delta.
- **Power BI** — connects to Databricks SQL Warehouses for dashboards.
- **Event Hubs / Kafka** — streaming sources for Structured Streaming.

### Code

```python
# Access ADLS Gen2 via service principal (OAuth)
configs = {
  "fs.azure.account.auth.type": "OAuth",
  "fs.azure.account.oauth.provider.type":
     "org.apache.hadoop.fs.azurebfs.oauth2.ClientCredsTokenProvider",
  "fs.azure.account.oauth2.client.id": dbutils.secrets.get("kv","sp-id"),
  "fs.azure.account.oauth2.client.secret": dbutils.secrets.get("kv","sp-secret"),
  "fs.azure.account.oauth2.client.endpoint":
     "https://login.microsoftonline.com/<tenant>/oauth2/token"
}
df = spark.read.format("delta").load(
    "abfss://data@acct.dfs.core.windows.net/sales")
```

### Interview Q&A

**Q: How do you connect Databricks to ADLS Gen2?**
Preferably via **Unity Catalog external locations/storage credentials**, or a **service principal with OAuth** / **managed identity**. Credentials come from **Key Vault–backed secret scopes**. Legacy approach is mounting with account keys/SAS (discouraged).

**Q: How do ADF and Databricks work together?**
**ADF orchestrates**: a pipeline ingests/copies data into ADLS, then triggers a **Databricks notebook/job activity** to transform it (often the medallion flow), and finally loads Gold data to Synapse/Power BI.

**Q: How do you serve Databricks data to BI?**
Expose curated **Gold** Delta tables via a **Databricks SQL Warehouse** and connect **Power BI** (or use Delta Sharing / the Synapse connector).

---

## 16. Common Coding / Scenario Tasks

### 1. Read CSV → write Delta (Bronze)

```python
df = (spark.read.option("header", True).option("inferSchema", True)
            .csv("/mnt/landing/sales.csv"))
df.write.format("delta").mode("overwrite").saveAsTable("bronze.sales")
```

### 2. MERGE / Upsert (SCD Type 1)

```sql
MERGE INTO silver.customers t
USING staging.customers s ON t.id = s.id
WHEN MATCHED THEN UPDATE SET *
WHEN NOT MATCHED THEN INSERT *;
```

### 3. SCD Type 2 (track history)

```sql
MERGE INTO dim_customer t
USING updates s ON t.id = s.id AND t.is_current = true
WHEN MATCHED AND t.attr <> s.attr THEN
  UPDATE SET t.is_current = false, t.end_date = current_date()
WHEN NOT MATCHED THEN
  INSERT (id, attr, is_current, start_date) VALUES (s.id, s.attr, true, current_date());
```

### 4. Deduplicate keeping latest record

```python
from pyspark.sql.window import Window
from pyspark.sql import functions as F
w = Window.partitionBy("id").orderBy(F.col("updated_at").desc())
(df.withColumn("rn", F.row_number().over(w))
   .filter("rn = 1").drop("rn")
   .write.format("delta").mode("overwrite").saveAsTable("silver.events"))
```

### 5. Incremental ingest with Auto Loader

```python
(spark.readStream.format("cloudFiles")
   .option("cloudFiles.format", "parquet")
   .option("cloudFiles.schemaLocation", "/chk/schema")
   .load("/mnt/landing")
 .writeStream.option("checkpointLocation", "/chk/cp")
   .trigger(availableNow=True).toTable("bronze.raw"))
```

### 6. Time travel / rollback

```sql
SELECT * FROM sales VERSION AS OF 10;
RESTORE TABLE sales TO VERSION AS OF 10;
```

### 7. Optimize + Z-Order + Vacuum

```sql
OPTIMIZE sales ZORDER BY (customer_id);
VACUUM sales RETAIN 168 HOURS;
```

### 8. Gold aggregation

```python
(spark.table("silver.sales")
   .groupBy("region").agg(F.sum("amount").alias("total"))
   .write.format("delta").mode("overwrite").saveAsTable("gold.sales_by_region"))
```

### 9. Read a secret & access ADLS

```python
key = dbutils.secrets.get("kv-scope", "adls-key")
spark.conf.set("fs.azure.account.key.acct.dfs.core.windows.net", key)
```

### 10. Parameterized notebook

```python
dbutils.widgets.text("load_date", "")
load_date = dbutils.widgets.get("load_date")
df.filter(F.col("date") == load_date)
```

---

## 17. Tricky / Gotcha Questions

**1. Control plane never stores your data.** Data always lives in the **data plane** (your storage). A common misconception.

**2. `DROP TABLE` on a managed table deletes the data files**; on an external table it does not.

**3. VACUUM with low retention breaks time travel.** Default retention is **7 days**; going lower removes the files older versions depend on (and Databricks blocks it without overriding a safety flag).

**4. All-purpose clusters cost more than job clusters.** Use **job clusters** for scheduled production work.

**5. Auto-termination ≠ autoscaling.** Auto-termination **shuts down idle clusters**; autoscaling **adjusts worker count** while running.

**6. Unity Catalog uses a 3-level namespace** (`catalog.schema.table`) — legacy Hive uses 2-level. Queries written for one can break on the other.

**7. Delta is Parquet + a log.** Reading the Parquet files directly bypasses the transaction log and can return **stale/incorrect** data — always read as Delta.

**8. Schema enforcement blocks mismatched appends** unless you opt into **schema evolution** (`mergeSchema`).

**9. `display()` is Databricks-only** (rich rendering); plain Spark uses `show()`.

**10. Streaming needs a checkpoint location** — without it you lose exactly-once/restart guarantees.

**11. Mounts are global and credentials-embedded** — under Unity Catalog prefer **Volumes / external locations** for governed access.

**12. OPTIMIZE doesn't delete old files** — it rewrites/compacts; you still need **VACUUM** to reclaim storage.

---

## 18. Rapid-Fire One-Liners

- **Azure Databricks** = managed, Azure-native Spark **lakehouse** platform.
- **Lakehouse** = data lake storage + warehouse ACID/performance (via Delta).
- **Control plane** (Databricks) = management/metadata; **data plane** (your sub) = compute + data.
- **All-purpose cluster** = interactive, costlier; **job cluster** = ephemeral, production.
- **Autoscaling** adjusts workers; **auto-termination** kills idle clusters.
- **Delta Lake** = Parquet + **`_delta_log`** transaction log.
- **ACID, time travel, schema evolution, MERGE** = Delta's headline features.
- **Time travel:** `VERSION AS OF` / `TIMESTAMP AS OF`; `RESTORE` to roll back.
- **OPTIMIZE** compacts files; **ZORDER** co-locates for data skipping; **VACUUM** cleans old files.
- **Managed table** drop = data deleted; **external** drop = data kept.
- **Medallion:** Bronze (raw) → Silver (clean) → Gold (curated/aggregated).
- **Unity Catalog** = central governance, 3-level namespace, row/column security, lineage.
- **Delta Sharing** = open cross-org data sharing without copying.
- **DLT** = declarative ETL with expectations + auto orchestration.
- **Auto Loader (`cloudFiles`)** = incremental file ingestion.
- **Photon** = vectorized native engine for speed.
- **AQE** = runtime query re-optimization.
- **Secrets** via `dbutils.secrets.get` (Key Vault–backed scope).
- **DBU** = billing unit; cluster type/size drives cost.
- **Volumes** = governed file storage (replacing DBFS mounts under UC).
- **ADF** orchestrates; **Power BI**/Synapse consume Gold.

---

## 19. Last-Minute Checklist

Run through these the hour before your interview:

- [ ] What a lakehouse is; Databricks vs plain Spark.
- [ ] Control plane vs data plane (where data lives).
- [ ] All-purpose vs job cluster; autoscaling vs auto-termination; DBR (ML/Photon).
- [ ] Delta Lake = Parquet + `_delta_log`; ACID via optimistic concurrency.
- [ ] Time travel, schema enforcement vs evolution, OPTIMIZE/ZORDER/VACUUM, Liquid Clustering.
- [ ] Managed vs external tables (drop behavior).
- [ ] Medallion architecture (Bronze/Silver/Gold) and what each layer does.
- [ ] Unity Catalog: 3-level namespace, fine-grained grants, lineage, Delta Sharing, Volumes.
- [ ] Delta Live Tables vs notebooks; expectations.
- [ ] Workflows/Jobs orchestration; why job clusters.
- [ ] Auto Loader + Structured Streaming + checkpoints (exactly-once).
- [ ] Optimization levers: small-file problem, data skipping, broadcast, Photon, AQE.
- [ ] Secrets/Key Vault; Entra ID security; ADLS Gen2 access patterns.
- [ ] Azure integration: ADF, Synapse, Power BI, Event Hubs.
- [ ] Be ready to write: read→Delta, MERGE/upsert, SCD2, dedupe-latest, Auto Loader, time travel, OPTIMIZE.

**Interview tips:** Frame answers around the **lakehouse + Delta + medallion** story — it's the backbone of most Databricks roles. When asked about performance, lead with **OPTIMIZE/Z-Order, data skipping, Photon, and minimizing shuffles**. When asked about governance, lead with **Unity Catalog**. For production design, emphasize **job clusters, DLT/Workflows, Auto Loader, and Key Vault secrets**.

---

*Good luck, Tejas — read it top-to-bottom once, then drill Sections 5–9 (Delta, medallion, Unity Catalog) and Section 16 (write each snippet from memory). Those are the heart of every Databricks interview.*
