# Data Engineering Concepts — Last-Minute Interview Prep Guide

> **How to use this:** Skim the **Theory** for crisp definitions, glance at the **Program/Example** to see it concretely, and rehearse the **Interview Q&A** out loud. Bold lines are the one-liners interviewers want to hear. This is **platform-agnostic core DE theory** — the concepts asked regardless of whether you're on Azure, AWS, GCP, Spark, or Snowflake. Built for a 2–4 hour final revision.

---

## Table of Contents

1. [The Data Engineer Role & Data Lifecycle](#1-the-data-engineer-role--data-lifecycle)
2. [ETL vs ELT](#2-etl-vs-elt)
3. [Batch vs Streaming Processing](#3-batch-vs-streaming-processing)
4. [OLTP vs OLAP](#4-oltp-vs-olap)
5. [Data Warehouse vs Data Lake vs Lakehouse](#5-data-warehouse-vs-data-lake-vs-lakehouse)
6. [Data Modeling & Dimensional Modeling](#6-data-modeling--dimensional-modeling)
7. [Slowly Changing Dimensions (SCD)](#7-slowly-changing-dimensions-scd)
8. [Normalization vs Denormalization](#8-normalization-vs-denormalization)
9. [File Formats (Parquet, Avro, ORC, CSV)](#9-file-formats)
10. [Partitioning & Bucketing](#10-partitioning--bucketing)
11. [Open Table Formats (Delta, Iceberg, Hudi)](#11-open-table-formats)
12. [Distributed Processing & Shuffles](#12-distributed-processing--shuffles)
13. [Incremental Loads & CDC](#13-incremental-loads--cdc)
14. [Idempotency & Exactly-Once](#14-idempotency--exactly-once)
15. [Schema Evolution & Data Contracts](#15-schema-evolution--data-contracts)
16. [Data Quality](#16-data-quality)
17. [Orchestration & Workflow Management](#17-orchestration--workflow-management)
18. [Data Governance, Lineage & Observability](#18-data-governance-lineage--observability)
19. [SQL Essentials for DE](#19-sql-essentials-for-de)
20. [System Design for Data Pipelines](#20-system-design-for-data-pipelines)
21. [Common Coding Programs](#21-common-coding-programs)
22. [Tricky / Gotcha Questions](#22-tricky--gotcha-questions)
23. [Rapid-Fire One-Liners](#23-rapid-fire-one-liners)
24. [Last-Minute Checklist](#24-last-minute-checklist)

---

## 1. The Data Engineer Role & Data Lifecycle

### Theory

A **data engineer** designs, builds, and maintains the systems that **collect, store, transform, and serve** data so analysts, scientists, and applications can use it reliably.

**The data engineering lifecycle (Reis/Housley model):**
**Generation** (sources) → **Ingestion** → **Storage** → **Transformation** → **Serving** (analytics/ML/reverse-ETL).

Cutting across all stages are the **undercurrents**: **security, data management/governance, DataOps, data architecture, orchestration, and software engineering**.

**Core responsibilities:** build pipelines (batch + streaming), model data, ensure data quality and reliability, optimize cost/performance, manage orchestration, and enforce governance/security.

### Interview Q&A

**Q: What does a data engineer do vs a data scientist/analyst?**
A **data engineer** builds and maintains the **data infrastructure and pipelines** that deliver clean, reliable, well-modeled data. A **data scientist** builds models on that data; an **analyst** queries it for insights. DEs make the data *trustworthy and accessible*; the others *consume* it.

**Q: Walk me through a data pipeline you'd design end-to-end.**
Source → **ingest** (batch/stream, full/incremental) → **land raw** in storage (lake/lakehouse Bronze) → **transform/clean/model** (Silver/Gold) → **serve** to a warehouse/BI/ML. Add **orchestration, data-quality checks, monitoring/alerting, security, and cost controls** throughout.

**Q: What makes a pipeline "production-grade"?**
**Reliability** (idempotent, restartable, monitored), **data quality** (validated), **scalability**, **observability** (logging, lineage, alerts), **security/governance**, and **cost-efficiency** — not just "it runs."

---

## 2. ETL vs ELT

### Theory

- **ETL (Extract, Transform, Load):** transform data **in a staging area before** loading into the target. Traditional; good when you must clean/mask before landing (e.g., PII) or the target has limited compute.
- **ELT (Extract, Load, Transform):** load **raw** data into the target first, then transform **inside** the powerful target (cloud warehouse / lakehouse). **The modern default** — leverages elastic cloud compute, keeps raw data for reprocessing, and fits the medallion pattern.

### Program (conceptual)

```text
ETL:  Source → [Extract] → [Transform in staging] → [Load clean data] → Warehouse
ELT:  Source → [Extract] → [Load raw] → Warehouse/Lake → [Transform in place] → curated
```

### Interview Q&A

**Q: ETL vs ELT — which and why?**
**ELT** is the modern default in cloud data engineering: load raw data first, then transform using the warehouse/lakehouse's scalable compute. It's cheaper, more flexible, preserves raw data for re-processing, and fits medallion architectures. **ETL** still suits cases requiring transformation *before* storage — e.g., PII masking/compliance or limited target compute.

**Q: Why keep raw data in ELT?**
For **reprocessing and auditability** — if business logic changes or a bug is found, you can rebuild downstream layers from untouched raw data without re-extracting from source.

---

## 3. Batch vs Streaming Processing

### Theory

- **Batch:** process a **bounded** set of data on a **schedule** (hourly/daily). High throughput, higher latency, simpler, cheaper. Use for reports, daily loads, backfills.
- **Streaming:** process **unbounded** data **as events arrive** (sub-second to seconds latency). Use for real-time dashboards, fraud detection, alerting, IoT.
- **Micro-batch** (e.g., Spark Structured Streaming): small batches at short intervals — a middle ground.

**Key streaming concepts:** **event time vs processing time**, **windowing** (tumbling/sliding/session), **watermarks** (handle late data), **checkpointing** (fault tolerance / exactly-once).

**Lambda vs Kappa architecture:** **Lambda** runs separate batch + speed layers (complex, duplicated logic). **Kappa** uses a single streaming layer for everything (reprocess by replaying the log).

### Interview Q&A

**Q: Batch vs streaming — when do you choose each?**
**Batch** when latency tolerance is high and data is bounded (daily reports, large reprocessing) — simpler and cheaper. **Streaming** when you need **low-latency, real-time** results (fraud, monitoring, live dashboards). The decision is driven by the **latency SLA** and cost.

**Q: Event time vs processing time?**
**Event time** is when the event actually occurred; **processing time** is when the system processes it. Out-of-order/late data means they differ — you use **watermarks** to decide how long to wait for late events before finalizing a window.

**Q: What is a watermark in streaming?**
A threshold on event time that tells the engine "I won't wait for data older than this" — it lets windows close and bounds state, handling **late-arriving data** gracefully.

**Q: Lambda vs Kappa architecture?**
**Lambda** maintains parallel **batch + streaming** layers (accurate but duplicated logic). **Kappa** uses a **single streaming** pipeline and reprocesses by **replaying** the event log — simpler, now common with durable logs like Kafka.

---

## 4. OLTP vs OLAP

### Theory

| | OLTP | OLAP |
|---|---|---|
| Purpose | Transactional (operational apps) | Analytical (reporting/BI) |
| Workload | Many small reads/writes | Few large, complex reads |
| Schema | Normalized (3NF) | Denormalized (star/snowflake) |
| Examples | Azure SQL, PostgreSQL, MySQL | Snowflake, BigQuery, Synapse, Redshift |
| Optimized for | Row-level integrity, low latency writes | Aggregations, scans, joins |

### Interview Q&A

**Q: OLTP vs OLAP?**
**OLTP** systems handle high-volume **transactional** operations with normalized schemas and fast row-level reads/writes (operational databases). **OLAP** systems handle **analytical** queries — large aggregations over denormalized, columnar data (data warehouses). DEs typically move data **from OLTP sources into OLAP targets**.

**Q: Why not run analytics directly on the OLTP database?**
Heavy analytical scans would **degrade transactional performance**, and the **normalized schema** is poorly suited for aggregations. We offload to an OLAP warehouse/lakehouse modeled for analytics.

---

## 5. Data Warehouse vs Data Lake vs Lakehouse

### Theory

- **Data Warehouse:** structured, schema-on-write, optimized for SQL analytics/BI. Reliable & fast but rigid and pricier for raw/unstructured data (Snowflake, BigQuery, Synapse, Redshift).
- **Data Lake:** cheap object storage for **raw, any-format** data, schema-on-read. Flexible but risks becoming a "data swamp" without governance (S3, ADLS, GCS).
- **Lakehouse:** combines lake's **cheap open storage** with warehouse's **ACID, schema, performance** via **open table formats** (Delta/Iceberg/Hudi). The modern convergence (Databricks, Fabric).

**Schema-on-write** (warehouse: validate structure at load) vs **schema-on-read** (lake: apply structure at query).

### Interview Q&A

**Q: Data warehouse vs data lake vs lakehouse?**
A **warehouse** stores structured, modeled data for fast SQL/BI (schema-on-write). A **lake** stores cheap raw data in any format (schema-on-read) but needs governance. A **lakehouse** merges both — lake-cost open storage **plus** ACID transactions, schema enforcement, and warehouse-class performance via **Delta/Iceberg/Hudi**.

**Q: What problem does the lakehouse solve?**
It removes the need to maintain **two separate systems** (lake + warehouse) and copy data between them, giving one governed, ACID-compliant store for both data engineering and BI/ML on open formats.

**Q: Schema-on-read vs schema-on-write?**
**Schema-on-write** validates and structures data **at load time** (warehouse) — reliable, less flexible. **Schema-on-read** stores raw and applies schema **at query time** (lake) — flexible, but quality is enforced later.

---

## 6. Data Modeling & Dimensional Modeling

### Theory

**Data modeling** = structuring data to serve business needs efficiently.

**Dimensional modeling (Kimball):** organize analytical data into:

- **Fact tables** — measurable **business events/metrics** (sales amount, quantity) at a defined **grain**; mostly foreign keys + numeric measures.
- **Dimension tables** — **descriptive context** (customer, product, date, store) for filtering/grouping.

**Grain** = the level of detail one fact row represents (e.g., "one row per order line per day"). Define it first.

**Schemas:**
- **Star schema** — central fact + **denormalized** dimensions. Fewer joins, fast, BI-friendly.
- **Snowflake schema** — **normalized** dimensions (split into sub-tables). Less redundancy, more joins.
- **Galaxy/Fact constellation** — multiple fact tables sharing dimensions.

**Inmon vs Kimball:** Inmon = top-down, normalized enterprise warehouse then data marts. Kimball = bottom-up, dimensional marts first. **Data Vault** = a third approach (hubs/links/satellites) for auditable, scalable enterprise modeling.

**Surrogate key** = a system-generated key (e.g., integer) used instead of a natural/business key — stable, decoupled from source changes, enables SCD.

### Program (star schema)

```sql
-- Fact + dimensions (star)
CREATE TABLE dim_date    (date_key INT PK, date DATE, month INT, year INT);
CREATE TABLE dim_product (product_key INT PK, name STRING, category STRING);
CREATE TABLE dim_customer(customer_key INT PK, name STRING, region STRING);

CREATE TABLE fact_sales (
  sale_id BIGINT,
  date_key INT, product_key INT, customer_key INT,  -- FKs to dims
  quantity INT, amount DECIMAL(10,2)                  -- measures
);
```

### Interview Q&A

**Q: Star vs snowflake schema?**
**Star** has a central fact table with **denormalized dimensions** — fewer joins, faster queries, simpler for BI. **Snowflake** normalizes dimensions into sub-tables — saves storage and reduces redundancy but adds joins. Star is the usual default for analytics.

**Q: Fact vs dimension table?**
A **fact** table holds **measurable metrics/events** at a defined **grain**, mostly foreign keys and numbers. A **dimension** holds **descriptive attributes** used to slice and filter facts. Facts answer "how much," dimensions answer "who/what/when/where."

**Q: What is grain and why define it first?**
**Grain** is the level of detail of a fact row. Defining it first ensures all measures and dimensions are **consistent** with that level — mixing grains causes wrong aggregations.

**Q: Why use surrogate keys?**
They're **stable, system-generated** keys independent of source/natural keys — they survive source changes, enable **SCD Type 2** history, and improve join performance (integer keys).

**Q: Kimball vs Inmon vs Data Vault?**
**Kimball** = bottom-up dimensional marts (star schemas) — fast to deliver. **Inmon** = top-down normalized enterprise warehouse, then marts — strong integration. **Data Vault** = hubs/links/satellites for **auditable, agile** enterprise modeling with full history.

---

## 7. Slowly Changing Dimensions (SCD)

### Theory

**SCD** = how you handle **changes to dimension attributes** over time.

- **Type 0** — never changes (retain original).
- **Type 1** — **overwrite** the old value; no history.
- **Type 2** — **add a new row** for the changed version with `start_date`, `end_date`, and `is_current` flag (or version number). **Full history** — the most asked.
- **Type 3** — keep **limited history** in extra columns (e.g., `previous_value`).
- **Type 4** — move history to a separate history table.
- **Type 6** — hybrid of 1+2+3.

### Program (SCD Type 2 with MERGE)

```sql
MERGE INTO dim_customer t
USING staging s ON t.customer_id = s.customer_id AND t.is_current = true
WHEN MATCHED AND t.region <> s.region THEN
  UPDATE SET t.is_current = false, t.end_date = current_date()   -- expire old
WHEN NOT MATCHED THEN
  INSERT (customer_id, region, is_current, start_date)
  VALUES (s.customer_id, s.region, true, current_date());        -- insert new version
```

### Interview Q&A

**Q: Explain SCD Type 1 vs Type 2.**
**Type 1** overwrites the attribute — you only keep the current value, no history. **Type 2** preserves history by **inserting a new row** with validity columns (`start_date`, `end_date`, `is_current`), so you can see the value as of any point in time. Type 2 is implemented with a **MERGE**: expire the old row, insert the new version.

**Q: When would you use Type 2 vs Type 1?**
**Type 2** when historical accuracy matters (e.g., a customer's region at the time of each sale, for correct historical reporting). **Type 1** when you only care about the current value (e.g., correcting a typo) and history isn't needed.

---

## 8. Normalization vs Denormalization

### Theory

- **Normalization** — organize data to **minimize redundancy** and anomalies (1NF atomic values; 2NF no partial dependency; 3NF no transitive dependency). Used in **OLTP**.
- **Denormalization** — deliberately add redundancy to **reduce joins and speed reads**. Used in **OLAP/warehouses** (star schemas).

### Interview Q&A

**Q: 1NF, 2NF, 3NF in brief?**
**1NF:** atomic values, no repeating groups. **2NF:** 1NF + every non-key column depends on the *whole* primary key (no partial dependency). **3NF:** 2NF + no transitive dependencies (non-keys depend only on the key).

**Q: When normalize vs denormalize?**
**Normalize** for transactional systems where integrity and write efficiency matter. **Denormalize** for analytics where read performance and fewer joins matter — accept redundancy for speed.

---

## 9. File Formats

### Theory

| Format | Type | Best for | Notes |
|---|---|---|---|
| **CSV/JSON** | Row, text | Interchange, small data | No schema, no compression, slow scans |
| **Avro** | Row, binary | **Write-heavy, streaming**, schema evolution | Stores schema; good for Kafka |
| **Parquet** | **Columnar**, binary | **Analytics/read-heavy** | Compression, column pruning, predicate pushdown |
| **ORC** | Columnar, binary | Analytics (Hive ecosystem) | Similar to Parquet, good compression |

**Row vs columnar:** **row** formats (Avro) are efficient for writing whole records and streaming; **columnar** formats (Parquet/ORC) are efficient for **analytical reads** — scan only needed columns, compress well per column.

### Interview Q&A

**Q: Why Parquet over CSV for analytics?**
Parquet is **columnar, compressed, and carries a schema**, enabling **column pruning** and **predicate pushdown** — queries read far less data than row-based CSV/JSON, cutting I/O, time, and cost.

**Q: Avro vs Parquet — when each?**
**Avro** (row-based) for **write-heavy/streaming** and where **schema evolution** matters (e.g., Kafka messages). **Parquet** (columnar) for **read-heavy analytics**. A common pattern: Avro for ingestion/landing, Parquet/Delta for the analytical layers.

**Q: What is predicate pushdown?**
Pushing filter conditions **down to the storage/scan layer** so only matching data is read — columnar formats + statistics (min/max per block) let the engine **skip** non-matching chunks.

---

## 10. Partitioning & Bucketing

### Theory

- **Partitioning** — physically split a table/dataset into folders by a **column value** (usually **date**). Queries filtering on that column do **partition pruning** — reading only relevant partitions, slashing data scanned and cost.
- **Bucketing (clustering)** — hash data into a **fixed number of buckets** by a column; co-locates matching keys to speed **joins and aggregations** without a shuffle.

**Pitfalls:** **over-partitioning** (too many tiny files/partitions = metadata overhead, the **small-file problem**); **partition skew** (uneven sizes); choosing a **high-cardinality** partition column (bad).

### Program

```sql
-- Partitioned write (Spark/Delta)
df.write.partitionBy("year", "month").format("delta").save("/data/sales")

-- Query prunes to one partition
SELECT * FROM sales WHERE year = 2025 AND month = 6;   -- reads only that path
```

### Interview Q&A

**Q: What is partitioning and why does it help?**
Splitting data into segments by a column (often date) so queries **only scan relevant partitions** (partition pruning) — less data read, faster and cheaper. Choose a **low-cardinality**, frequently-filtered column.

**Q: Partitioning vs bucketing?**
**Partitioning** divides data into directories by column value for **pruning** on filters. **Bucketing** hashes rows into a fixed number of files by a key to optimize **joins/aggregations** (co-locating keys, avoiding shuffles). They're complementary.

**Q: What's the small-file problem and how do you avoid it?**
Many tiny files (from over-partitioning or frequent streaming appends) cause excessive metadata and slow reads. Avoid by **right-sizing partitions**, **compaction** (`OPTIMIZE`), and optimized/auto-compacting writes.

**Q: What happens if you partition on a high-cardinality column?**
You get **too many partitions/tiny files** (e.g., partition by user_id) — huge metadata overhead and slow queries. Partition on coarse, low-cardinality columns like date; use clustering/bucketing for finer access.

---

## 11. Open Table Formats

### Theory

**Open table formats** add a **transaction/metadata layer** over Parquet files to bring database features to the lake — the foundation of the **lakehouse**.

| Format | Origin | Highlights |
|---|---|---|
| **Delta Lake** | Databricks | Transaction log (`_delta_log`), ACID, time travel, MERGE, mature on Spark |
| **Apache Iceberg** | Netflix | Hidden partitioning, schema/partition evolution, engine-agnostic, snapshots |
| **Apache Hudi** | Uber | Upserts, incremental pulls, CoW/MoR storage, streaming-friendly |

**Common features:** ACID transactions, time travel/snapshots, schema evolution, upserts/deletes (MERGE), and scalable metadata.

### Interview Q&A

**Q: What are open table formats and why do they matter?**
Layers like **Delta, Iceberg, and Hudi** add **ACID transactions, time travel, schema evolution, and upserts** on top of Parquet in object storage — turning a raw data lake into a reliable **lakehouse** without a separate warehouse.

**Q: Delta vs Iceberg vs Hudi (high level)?**
**Delta** — tightly integrated with Spark/Databricks, simple and mature. **Iceberg** — engine-agnostic with strong **partition/schema evolution** and hidden partitioning. **Hudi** — optimized for **upserts and incremental** processing (CDC/streaming). All provide ACID + time travel; choice depends on ecosystem and workload.

**Q: How do these provide ACID on a lake?**
Through a **transaction log / metadata layer** that records atomic commits and snapshots, giving **snapshot isolation** for readers and atomic, consistent writes — even on cheap object storage.

---

## 12. Distributed Processing & Shuffles

### Theory

Big-data engines (Spark) process data in **partitions** across a cluster in parallel.

- **Transformations** are **lazy**; **actions** trigger execution (a DAG).
- **Narrow** transformations (map, filter) = no data movement. **Wide** transformations (groupBy, join, distinct) = **shuffle** (data redistributed across the network) — the main cost.
- **Data skew** — uneven key distribution overloads some tasks; fix with **salting**, broadcast joins, or adaptive execution.
- **Broadcast join** — send a small table to all nodes to avoid shuffling the large one.

**CAP theorem** (distributed systems): you can guarantee at most **two** of **Consistency, Availability, Partition tolerance**; since network partitions happen, you trade C vs A.

### Interview Q&A

**Q: What is a shuffle and why is it expensive?**
A **shuffle** redistributes data across partitions/nodes (for joins, groupBy, etc.), incurring **disk I/O, network transfer, and serialization**. Minimizing shuffles is the top performance lever in distributed processing.

**Q: How do you handle data skew?**
**Salting** (add a random prefix to spread a hot key), **broadcasting** the small side of a join, **repartitioning**, and enabling **adaptive query execution** to split skewed partitions at runtime.

**Q: Explain the CAP theorem.**
In a distributed system you can't simultaneously guarantee **Consistency, Availability, and Partition tolerance** — at most two. Because partitions are unavoidable, real systems trade **consistency vs availability** (e.g., CP vs AP databases).

**Q: What's a broadcast join?**
When one table is small enough to fit in memory, it's **copied to every node** so the large table isn't shuffled — turning an expensive shuffle join into a fast local join.

---

## 13. Incremental Loads & CDC

### Theory

**Don't reprocess everything** — load only new/changed data.

- **Full load** — reload the entire dataset (simple, expensive, fine for small/dimension tables).
- **Incremental load** — load only deltas, via:
  - **Watermark/high-water-mark** — track max `last_modified`/`id`, load rows greater than it.
  - **CDC (Change Data Capture)** — capture inserts/updates/deletes from the source's **transaction log** (e.g., Debezium, SQL CDC) — low-impact, near-real-time.
  - **Snapshot diff / hashing** — compare current vs previous to detect changes.

Apply deltas with **MERGE/upsert** (idempotent).

### Interview Q&A

**Q: How do you implement incremental loading?**
Track a **watermark** (max timestamp/ID of the last load), extract only rows newer than it, and **MERGE** them into the target; then update the watermark. For databases, **CDC** captures row-level changes from the transaction log for near-real-time, low-impact sync.

**Q: What is CDC and why use it?**
**Change Data Capture** reads the source database's **transaction log** to capture every insert/update/delete with minimal load on the source. It enables **near-real-time, complete** replication (including deletes) without full reloads or heavy queries.

**Q: Full load vs incremental — trade-offs?**
**Full** is simple and self-correcting but expensive and slow at scale. **Incremental** is efficient but more complex (needs reliable change tracking and idempotent merges). Use full for small/dimension tables, incremental for large fact tables.

---

## 14. Idempotency & Exactly-Once

### Theory

**Idempotency** — running the same operation **multiple times produces the same result** (no duplicates/corruption). Essential because pipelines retry and re-run.

**How to achieve it:** **MERGE/upsert on a key**, **overwrite a partition** instead of append, **deduplicate** on a unique id, and use **deterministic** keys/output paths.

**Delivery semantics:** **at-most-once** (may lose data), **at-least-once** (may duplicate), **exactly-once** (no loss, no dup — hardest; achieved via checkpoints + idempotent/transactional sinks).

### Program

```python
# Idempotent upsert instead of blind append
MERGE INTO target t USING batch s ON t.id = s.id
WHEN MATCHED THEN UPDATE SET *
WHEN NOT MATCHED THEN INSERT *
# Re-running the same batch won't create duplicates
```

### Interview Q&A

**Q: What is idempotency and why does it matter in pipelines?**
An operation is **idempotent** if re-running it yields the same result without duplicates. It's critical because jobs **fail and retry** — idempotent design (MERGE, partition overwrite, dedup) means a rerun is safe and won't corrupt data.

**Q: How do you guarantee exactly-once processing?**
Combine **checkpointing** (track processed offsets/state) with **idempotent or transactional sinks** (e.g., Delta atomic commits, MERGE on a key). On failure, the job resumes from the checkpoint and the idempotent write prevents duplicates.

**Q: How do you make a non-idempotent append idempotent?**
Replace blind `INSERT/append` with **MERGE on a business key**, or **overwrite the target partition** for the batch, or **dedupe** on a unique id after load.

---

## 15. Schema Evolution & Data Contracts

### Theory

- **Schema evolution** — handling changes to data structure over time (new columns, type changes) without breaking pipelines. Open formats support adding columns via `mergeSchema`/evolution; some support safe type widening.
- **Schema drift** — unexpected source schema changes; handle with evolution, **quarantine** of bad records, and alerts.
- **Data contracts** — a **formal agreement** between data producers and consumers defining schema, semantics, quality, and SLAs — shifting quality "left" to the source and preventing breaking changes.

### Interview Q&A

**Q: How do you handle schema changes/drift in a pipeline?**
Use formats/tables that support **schema evolution** (add columns safely), validate incoming schema, **quarantine** non-conforming records rather than failing the whole load, and **alert**. For controlled change, use **data contracts** so producers can't silently break consumers.

**Q: What is a data contract?**
A **formal, versioned agreement** between data producers and consumers specifying the schema, field semantics, quality expectations, and SLAs. It pushes data-quality responsibility **upstream** and prevents breaking changes from reaching downstream pipelines.

**Q: Backward vs forward compatible schema changes?**
**Backward compatible** = new schema can read old data (e.g., adding an optional column). **Forward compatible** = old schema can read new data. Safe evolution favors **additive, optional** changes; dropping/renaming/retyping columns is breaking.

---

## 16. Data Quality

### Theory

**Data quality dimensions:** **accuracy, completeness, consistency, timeliness/freshness, uniqueness, validity.**

**Common checks:** nulls in required fields, **duplicates**, referential integrity (orphan keys), value ranges/domains, **row-count anomalies**, and **freshness** (is the data recent?).

**Approach:** validate at ingestion and between layers; **quarantine** bad records; **monitor and alert** on failures; track quality metrics over time. Tools/patterns: **Great Expectations**, **dbt tests**, **Delta Live Tables expectations**.

### Program

```python
# Simple quality checks
assert df.filter(F.col("id").isNull()).count() == 0, "Null IDs found"
dupes = df.groupBy("id").count().filter("count > 1").count()
assert dupes == 0, f"{dupes} duplicate IDs"
# Freshness
max_ts = df.agg(F.max("event_time")).collect()[0][0]
assert max_ts >= yesterday, "Data is stale"
```

### Interview Q&A

**Q: How do you ensure data quality in a pipeline?**
Run automated checks for **nulls, duplicates, referential integrity, value ranges, row-count anomalies, and freshness** at ingestion and between layers. **Quarantine** bad records, **monitor and alert** on breaches, and track quality metrics over time. Frameworks like **Great Expectations / dbt tests / DLT expectations** formalize this.

**Q: What are the dimensions of data quality?**
**Accuracy, completeness, consistency, timeliness/freshness, uniqueness, and validity.** A good answer covers checks for each plus monitoring and alerting.

**Q: A downstream report shows wrong numbers. How do you debug?**
Trace **lineage** backward, check **row counts and quality metrics** at each layer, look for recent **schema/source changes**, verify **join logic and grain** (fan-out/duplication), and check the **latest successful load/freshness**. Reproduce with a known-good slice.

---

## 17. Orchestration & Workflow Management

### Theory

**Orchestration** = scheduling, sequencing, and monitoring pipeline tasks with **dependencies, retries, and alerting**.

- **Apache Airflow** — Python **DAGs**, mature, huge ecosystem; schedule + dependencies + sensors + backfills.
- **Dagster** — asset/data-aware orchestration, strong typing/testing, lineage-first.
- **Prefect** — Pythonic, dynamic, lighter-weight.
- Cloud-native: **ADF**, **AWS Step Functions/Glue**, **GCP Composer (managed Airflow)**, **Databricks Workflows**.

**Key concepts:** **DAG** (Directed Acyclic Graph), idempotent tasks, retries/timeouts, **backfills**, **sensors** (wait for an event), **XComs** (pass data), SLAs.

### Interview Q&A

**Q: What is a DAG and why is it used for pipelines?**
A **Directed Acyclic Graph** represents tasks and their **dependencies** with no cycles. Orchestrators run tasks in dependency order, in parallel where possible, with **retries, scheduling, and monitoring** — making complex pipelines reliable and observable.

**Q: Airflow vs Dagster vs Prefect?**
**Airflow** is the mature, widely-adopted standard (task-centric DAGs, big ecosystem). **Dagster** is **asset/data-aware** with strong typing, testing, and lineage. **Prefect** is lightweight and Pythonic with dynamic flows. Choice depends on team needs; Airflow is the safe default answer.

**Q: How do you handle a failed task in orchestration?**
**Retries with backoff + timeouts**, **alerting** on failure, **idempotent** tasks so reruns are safe, and dependency handling so downstream tasks don't run on bad data. For time-sliced loads, **backfill** the failed window.

**Q: Why must orchestrated tasks be idempotent?**
Because orchestrators **retry and backfill** — a non-idempotent task would duplicate or corrupt data on rerun. Idempotency makes retries and reprocessing safe.

---

## 18. Data Governance, Lineage & Observability

### Theory

- **Data governance** — policies for **security, access control, compliance (GDPR/PII), and data ownership/cataloging**. Tools: Unity Catalog, Microsoft Purview, Collibra.
- **Data lineage** — tracking data's **origin and transformations** end-to-end (table/column level) — vital for debugging, impact analysis, and compliance.
- **Data observability** — monitoring pipeline/data **health**: freshness, volume, schema, distribution, and lineage, with alerting (the "5 pillars": freshness, volume, schema, quality/distribution, lineage).
- **SLA/SLO/SLI** — **SLA** = the promise (e.g., "data ready by 8 AM"); **SLO** = the internal target; **SLI** = the measured indicator.

### Interview Q&A

**Q: What is data lineage and why does it matter?**
**Lineage** tracks where data comes from and how it's transformed across the pipeline, at table/column level. It's essential for **debugging quality issues, impact analysis** (what breaks if I change this), and **compliance/audit**.

**Q: What is data observability?**
Continuous monitoring of **data and pipeline health** — **freshness, volume, schema changes, distribution/quality, and lineage** — with alerting, so issues are caught **before** stakeholders notice. Like DevOps observability, applied to data.

**Q: SLA vs SLO vs SLI?**
**SLA** is the external **commitment** (e.g., "99.9% uptime / data by 8 AM"). **SLO** is the internal **target** you aim for. **SLI** is the actual **measured metric**. DEs define SLAs/SLOs for pipeline freshness and reliability.

**Q: How do you handle PII/sensitive data?**
**Classify** it, apply **access controls (RBAC), encryption, masking/tokenization**, **column/row-level security**, and audit logging — governed centrally (Unity Catalog/Purview) for compliance (GDPR, etc.).

---

## 19. SQL Essentials for DE

### Theory

SQL is the daily language of DE. Master: joins, aggregation, **window functions**, CTEs, subqueries, set operations, and query optimization (indexes, partition pruning, avoiding `SELECT *`).

### Program

```sql
-- Window functions: rank, running total, dedupe
SELECT *,
  ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date DESC) AS rn,
  SUM(amount)  OVER (PARTITION BY customer_id ORDER BY order_date)      AS running_total,
  LAG(amount)  OVER (PARTITION BY customer_id ORDER BY order_date)      AS prev_amount
FROM orders;

-- Nth highest
SELECT DISTINCT amount FROM orders ORDER BY amount DESC LIMIT 1 OFFSET 2;  -- 3rd highest

-- Dedupe keep latest (rn = 1 above)
-- Find duplicates
SELECT id, COUNT(*) FROM orders GROUP BY id HAVING COUNT(*) > 1;
```

### Interview Q&A

**Q: WHERE vs HAVING?**
**WHERE** filters rows **before** grouping (no aggregates). **HAVING** filters **groups after** `GROUP BY` (can use aggregates).

**Q: row_number vs rank vs dense_rank?**
**row_number** = unique sequential; **rank** = ties share a rank then **skip**; **dense_rank** = ties share a rank with **no gaps**.

**Q: How do you find/remove duplicates?**
Find: `GROUP BY key HAVING COUNT(*) > 1`. Remove keeping latest: `ROW_NUMBER() OVER (PARTITION BY key ORDER BY ts DESC)` and keep `rn = 1`.

**Q: How do you optimize a slow query?**
Check the **execution plan**, add **indexes** on filter/join columns, leverage **partition pruning**, avoid `SELECT *`, filter early, avoid functions on indexed columns in WHERE, and ensure join keys share types.

---

## 20. System Design for Data Pipelines

### Theory

A repeatable framework for "Design a data pipeline for X" questions:

1. **Requirements** — data volume, velocity (batch/stream), latency SLA, consumers, budget.
2. **Ingestion** — sources, full vs incremental, batch vs streaming, connectors/CDC.
3. **Storage** — lake/lakehouse zones (Bronze/Silver/Gold), file/table format, partitioning.
4. **Processing/Transform** — engine (Spark/SQL), modeling (star, SCD), data quality.
5. **Serving** — warehouse/BI/ML/API; access patterns.
6. **Orchestration** — DAG, scheduling, dependencies, retries.
7. **Cross-cutting** — idempotency, monitoring/observability, lineage, security/governance, **cost**.

### Interview Q&A

**Q: Design a pipeline to process millions of daily transactions for analytics.**
"Clarify SLA and volume. **Ingest** via CDC or batch into **Bronze** (raw, partitioned by date). **Transform** in Spark to **Silver** (clean, dedupe, schema-enforce) and **Gold** (star-schema aggregates, SCD2 dims) with **idempotent MERGE**. Serve **Gold** to the warehouse/BI. **Orchestrate** with Airflow/Workflows (retries, alerts). Add **data-quality checks, observability, lineage, security, and cost controls** (partition pruning, job clusters, columnar formats)."

**Q: Design a real-time pipeline (e.g., fraud detection).**
"**Stream** events through **Kafka**, process with a **streaming engine** (Spark Structured Streaming/Flink) using **windowing + watermarks**, apply rules/ML, and write results to a **fast store** for alerting — with **checkpointing for exactly-once**. Keep a **batch path or Kappa replay** for reprocessing, plus monitoring on lag/throughput."

**Q: How do you control cost in a pipeline?**
**Incremental** (not full) loads, **partition pruning + columnar formats** (scan less), **autoscaling/ephemeral compute** with auto-termination, **spot instances**, caching reused data, minimizing shuffles, and **lifecycle tiering** of cold data.

---

## 21. Common Coding Programs

### 1. Word count (the data "hello world")

```python
from collections import Counter
text = "the quick brown fox the lazy fox"
print(Counter(text.split()))   # the:2, fox:2, ...
```

### 2. Deduplicate keeping latest (PySpark)

```python
from pyspark.sql.window import Window
from pyspark.sql import functions as F
w = Window.partitionBy("id").orderBy(F.col("updated_at").desc())
clean = df.withColumn("rn", F.row_number().over(w)).filter("rn = 1").drop("rn")
```

### 3. Second-highest value (SQL)

```sql
SELECT MAX(amount) FROM orders WHERE amount < (SELECT MAX(amount) FROM orders);
```

### 4. Running total / top-N per group (SQL)

```sql
SELECT * FROM (
  SELECT *, ROW_NUMBER() OVER (PARTITION BY dept ORDER BY salary DESC) rn
  FROM emp) t
WHERE rn <= 3;   -- top 3 per dept
```

### 5. Idempotent upsert (Delta/SQL)

```sql
MERGE INTO target t USING source s ON t.id = s.id
WHEN MATCHED THEN UPDATE SET *
WHEN NOT MATCHED THEN INSERT *;
```

### 6. Flatten nested JSON (PySpark)

```python
from pyspark.sql import functions as F
flat = df.select("id", F.explode("items").alias("item")) \
         .select("id", "item.*")
```

### 7. Detect duplicates (SQL)

```sql
SELECT id, COUNT(*) FROM table GROUP BY id HAVING COUNT(*) > 1;
```

### 8. Pivot (SQL)

```sql
SELECT region,
  SUM(CASE WHEN year=2024 THEN amount END) AS y2024,
  SUM(CASE WHEN year=2025 THEN amount END) AS y2025
FROM sales GROUP BY region;
```

---

## 22. Tricky / Gotcha Questions

**1. ELT vs ETL** — modern cloud default is **ELT** (transform in the warehouse). Don't reflexively say ETL.

**2. Partitioning ≠ bucketing.** Partition for **pruning on filters**; bucket for **joins/aggregations**.

**3. High-cardinality partition column = small-file disaster.** Partition on coarse columns (date), not user_id.

**4. Append is not idempotent.** Reruns duplicate data — use **MERGE** or partition overwrite.

**5. Star vs snowflake:** star = denormalized (fewer joins); snowflake = normalized (more joins).

**6. SCD Type 1 loses history; Type 2 keeps it.** Know which the question needs.

**7. CDC captures deletes; watermark queries usually don't.** Mention this when deletes matter.

**8. Exactly-once needs checkpoints + idempotent sink** — streaming alone doesn't guarantee it.

**9. Columnar (Parquet) for reads, row (Avro) for writes/streaming** — don't use CSV for analytics.

**10. Shuffles are the expensive part** of distributed jobs — minimize them.

**11. CAP theorem:** you trade **C vs A** under partitions — you can't have all three.

**12. Schema-on-read (lake) vs schema-on-write (warehouse)** — quality is enforced at different times.

---

## 23. Rapid-Fire One-Liners

- **DE lifecycle:** Generation → Ingestion → Storage → Transformation → Serving.
- **ETL** transforms before load; **ELT** loads raw then transforms (modern default).
- **Batch** = scheduled, bounded; **streaming** = real-time, unbounded.
- **OLTP** = transactional/normalized; **OLAP** = analytical/denormalized.
- **Warehouse** (structured, schema-on-write) vs **lake** (raw, schema-on-read) vs **lakehouse** (both).
- **Fact** = measures at a grain; **dimension** = descriptive context.
- **Star** = denormalized dims; **snowflake** = normalized dims.
- **SCD2** keeps history via new rows + validity flags.
- **Surrogate key** = stable system key enabling SCD/joins.
- **Parquet/ORC** columnar for reads; **Avro** row for writes/streaming.
- **Partitioning** prunes scans; **bucketing** speeds joins.
- **Delta/Iceberg/Hudi** = ACID + time travel + upserts on the lake (lakehouse).
- **Shuffle** = costly data redistribution; minimize it.
- **Skew** fixed by salting / broadcast / AQE.
- **CDC** captures inserts/updates/deletes from the txn log.
- **Idempotency** = safe reruns; use MERGE/partition overwrite.
- **Exactly-once** = checkpoints + idempotent/transactional sink.
- **Data contract** = producer–consumer schema/quality agreement.
- **DAG** = task dependency graph (Airflow/Dagster/Prefect).
- **Lineage** = data origin + transformations; **observability** = freshness/volume/schema/quality/lineage.
- **SLA** (promise) > **SLO** (target) > **SLI** (measured).
- **CAP:** pick 2 of Consistency/Availability/Partition-tolerance.

---

## 24. Last-Minute Checklist

The hour before:

- [ ] DE lifecycle and what makes a pipeline production-grade.
- [ ] ETL vs ELT; batch vs streaming (and when each).
- [ ] OLTP vs OLAP; warehouse vs lake vs lakehouse; schema-on-read/write.
- [ ] Dimensional modeling: fact/dim, grain, star vs snowflake, surrogate keys.
- [ ] **SCD Type 1 vs Type 2** (be ready to write the MERGE).
- [ ] Normalization (1NF/2NF/3NF) vs denormalization.
- [ ] File formats: Parquet/ORC vs Avro vs CSV; predicate pushdown.
- [ ] Partitioning vs bucketing; small-file problem; partition pruning.
- [ ] Open table formats (Delta/Iceberg/Hudi) and what they add.
- [ ] Distributed processing: shuffle, skew, broadcast, lazy/DAG; CAP theorem.
- [ ] Incremental loads + **CDC**; idempotency + exactly-once.
- [ ] Schema evolution/drift; data contracts.
- [ ] Data quality dimensions + checks + quarantine/alerting.
- [ ] Orchestration: DAGs, Airflow/Dagster/Prefect, retries, backfills.
- [ ] Governance, lineage, observability; SLA/SLO/SLI; PII handling.
- [ ] SQL: window functions, dedupe, top-N, optimization.
- [ ] Practice 2–3 **system-design** answers using the §20 framework.

**Interview tips:** For concept questions, give the **definition + when to use it + a trade-off** — that depth signals seniority. For design questions, always walk the **framework** (requirements → ingest → store → transform → serve → orchestrate → cross-cutting) and end with **idempotency, data quality, observability, and cost**. Tie answers to **real pipelines you've built** wherever possible — interviewers value applied experience over textbook recall.

---

*Good luck — read it top-to-bottom once, then drill the high-frequency hits: ETL/ELT, batch vs streaming, dimensional modeling + SCD2, partitioning, idempotency/CDC, and the system-design framework. Those come up in nearly every data engineering interview.*
