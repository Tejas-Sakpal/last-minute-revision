# Scenario-Based Interview — Last-Minute Prep Guide

> **What this is:** The **situational / "what would you do if…" questions** that dominate experienced data engineer interviews — split into **(A) resume-tailored scenarios** built around Tejas's projects (Talend migration, medallion, Spark optimization, ADF, MongoDB/Elasticsearch) and **(B) general scenarios** every DE faces. Each has a **structured, detailed answer** you can adapt.
>
> **How scenario questions are graded:** interviewers want your **thought process** — clarifying questions, trade-offs, and production awareness — not a memorized fact. Always: *clarify → reason → propose → mention trade-offs/monitoring/cost.*

---

## Table of Contents

**Part A — Resume-Tailored Scenarios**
1. [How to Answer Any Scenario Question (Framework)](#1-how-to-answer-any-scenario-question)
2. [Talend → Databricks Migration Scenarios](#2-talend--databricks-migration-scenarios)
3. [Medallion / Delta Pipeline Scenarios](#3-medallion--delta-pipeline-scenarios)
4. [Spark Performance & Optimization Scenarios](#4-spark-performance--optimization-scenarios)
5. [ADF Orchestration Scenarios](#5-adf-orchestration-scenarios)
6. [MongoDB / Elasticsearch / Azure SQL Scenarios](#6-mongodb--elasticsearch--azure-sql-scenarios)

**Part B — General Data Engineering Scenarios**
7. [Pipeline Failure & Recovery](#7-pipeline-failure--recovery)
8. [Incremental Load & CDC](#8-incremental-load--cdc)
9. [Data Skew & Shuffle](#9-data-skew--shuffle)
10. [Small Files & File Size](#10-small-files--file-size)
11. [Schema Drift & Evolution](#11-schema-drift--evolution)
12. [Data Quality & Bad Data](#12-data-quality--bad-data)
13. [Duplicates & Idempotency](#13-duplicates--idempotency)
14. [Streaming / Real-Time](#14-streaming--real-time)
15. [Cost Optimization](#15-cost-optimization)
16. [Security & Governance](#16-security--governance)
17. [CI/CD & Deployment](#17-cicd--deployment)
18. [System Design Scenarios](#18-system-design-scenarios)
19. [Behavioral Scenarios (STAR)](#19-behavioral-scenarios-star)
20. [Rapid-Fire Scenario Drills](#20-rapid-fire-scenario-drills)
21. [Last-Minute Checklist](#21-last-minute-checklist)

---

# PART A — RESUME-TAILORED SCENARIOS

## 1. How to Answer Any Scenario Question

### The Framework (say it in this order)

1. **Clarify** — ask 1–2 questions (data volume? latency SLA? batch or stream? one-off or recurring?). This alone signals seniority.
2. **Restate** the problem and your assumptions.
3. **Diagnose** (for "it's broken/slow" questions) — how you'd find root cause before fixing.
4. **Propose** a solution, step by step.
5. **Trade-offs** — alternatives and why you chose yours.
6. **Production concerns** — idempotency, monitoring, data quality, cost, security.

**Memory hook:** *Clarify → Diagnose → Propose → Trade-offs → Productionize.*

---

## 2. Talend → Databricks Migration Scenarios

### Scenario: "You're migrating 200 Talend jobs to Databricks. How do you approach it, and how do you guarantee correctness?"

**Answer:**
"First I'd **clarify** scope — how many jobs are active, their dependencies, and the cut-over deadline. Then a **phased, prioritized approach**, not big-bang:

1. **Inventory & categorize** jobs by complexity and business criticality; start with simpler, lower-risk ones to build a reusable pattern library.
2. **Re-engineer, don't lift-and-shift** — there's no one-click converter, so each Talend job is **re-implemented in PySpark/Spark SQL**, converting row-based `tMap`/lookup logic into **set-based joins, `when/otherwise`, and window functions**, with **broadcast joins** for small lookups.
3. **Validate with parallel runs** — run Talend and Databricks on the same input and reconcile **row counts, aggregate sums on key columns, and record-level checksums** on the primary key. Only cut over when parity passes.
4. **Re-platform orchestration** — Talend scheduling → **ADF triggers/dependencies** with retries and alerts.
5. **Land output on medallion Delta** for reliability.

**Trade-off:** phased migration takes longer but de-risks; big-bang is faster but dangerous. **Production:** I'd keep the Talend job as a fallback until the Databricks version runs clean for a few cycles."

### Scenario: "After migration, the Databricks output doesn't match Talend for some rows. How do you debug?"

**Answer:**
"I'd **isolate the mismatch**: filter to the differing keys and compare field-by-field. The usual culprits are **null handling** (Talend vs Spark semantics), **implicit type casting / date formats**, **rounding/decimal precision**, **trim/whitespace**, and **join cardinality** (a duplicate in a lookup causing fan-out). I'd reproduce on a small slice, fix the specific transformation, and re-run reconciliation. I'd also add the case to my **validation test suite** so it can't regress."

---

## 3. Medallion / Delta Pipeline Scenarios

### Scenario: "Design an enterprise pipeline that ingests daily source files, applies CDC, and serves BI. Walk me through it."

**Answer:**
"**Bronze** — ingest raw files as-is into append-only Delta (partitioned by load date) using **Auto Loader** for efficient incremental file pickup; keep full history for audit/reprocessing.
**Silver** — clean, dedupe, enforce schema, and apply **CDC logic with Delta `MERGE`** (insert new, update changed, optionally soft-delete) into conformed entities.
**Gold** — build **BI-ready aggregates / star schema**, optimized with **`OPTIMIZE` + `ZORDER`** on common filter columns.
**ADF** orchestrates the layer dependencies with retries and alerting; runs on **job clusters** for cost.
**Serving** — Gold tables exposed via SQL endpoint / Fabric Direct Lake to **Power BI**.
**Cross-cutting:** idempotent MERGE (safe reruns), data-quality checks between layers, monitoring/lineage, Key Vault secrets."

### Scenario: "A business user says a Gold metric is wrong. How do you trace it?"

**Answer:**
"I'd trace **lineage backward** from Gold → Silver → Bronze. Check: (1) **row counts** at each layer for unexpected drops/spikes; (2) **join logic and grain** in Gold — a fan-out from a duplicate in a dimension is the classic cause of inflated metrics; (3) **latest successful load / freshness** — maybe Gold ran on stale Silver; (4) recent **source or schema changes**. I'd reproduce with a known-good slice, fix the root cause, and **backfill** the affected partitions. Then add a **data-quality/reconciliation check** so it's caught automatically next time."

### Scenario: "Why keep a Bronze layer at all if it's just raw data?"

**Answer:**
"Because it's the **immutable source of truth**. If a transformation bug is found or business logic changes, I can **rebuild Silver and Gold from Bronze** without re-extracting from source — which may not even be possible if the source has changed. It's also essential for **auditing** and reprocessing historical data."

---

## 4. Spark Performance & Optimization Scenarios

### Scenario: "A Databricks notebook that used to run in 20 min now takes 2 hours. How do you investigate and fix it?"

**Answer:**
"**Diagnose first** via the **Spark UI** — I'd look at stage timelines for **high shuffle read/write**, **skewed tasks** (a few tasks running far longer), **spill to disk**, and the number of tasks/partitions. I'd read the physical plan with **`explain()`**. Common root causes and fixes:

- **Growing shuffle / data volume** → **broadcast** small join tables, reduce shuffled columns (select early), tune `spark.sql.shuffle.partitions`, enable **AQE**.
- **Data skew** → **salting** the hot key or AQE skew-join handling.
- **Small-file problem** (table grew with many tiny files) → **`OPTIMIZE`** / auto-compaction.
- **Lost cache / recompute** → cache reused DataFrames.
- **Under-provisioned cluster / no Photon** → right-size, enable **Photon**.

I'd change **one thing at a time**, measure, and confirm in the plan (e.g., a `BroadcastHashJoin` appearing). **Production:** add monitoring on job duration so regressions alert early."

### Scenario: "A join between a 1 TB fact table and a 50 MB dimension is very slow. Optimize it."

**Answer:**
"The 50 MB dimension is small enough to **broadcast** — that sends it to every executor and **eliminates the shuffle** of the 1 TB fact, which is the expensive part. I'd use `broadcast(dim)` (or rely on `autoBroadcastJoinThreshold`), **partition the fact on the join key**, **cache** the dimension if reused, and confirm a **BroadcastHashJoin** in `explain()`. If the dimension were too big to broadcast, I'd consider **bucketing both tables** on the join key to avoid the shuffle."

### Scenario: "Your Spark job keeps failing with out-of-memory errors. What do you check?"

**Answer:**
"OOM usually means **too much data per task or on the driver**. I'd check: (1) am I calling **`collect()`** on a large dataset (pulls everything to the driver)? — replace with `write`/`take`. (2) **Skew** concentrating data in few partitions → repartition/salt. (3) **Wide transformations** with huge shuffle → increase partitions. (4) Large **broadcast** exceeding memory → raise threshold or don't broadcast. (5) Right-size **executor memory** and avoid caching everything. The fix is usually **better partitioning + avoiding driver-side collection**, not just bigger machines."

---

## 5. ADF Orchestration Scenarios

### Scenario: "An ADF pipeline fails intermittently at 3 AM and someone manually reruns it every morning. How do you fix this for good?"

**Answer:**
"First, **make it self-healing**: add **retry policies with backoff and timeouts** on activities so transient failures (network/throttling) auto-recover. Second, **make it idempotent/restartable** — use **watermarks + `MERGE`** so a rerun doesn't duplicate data and can safely resume. Third, **alerting** — on-failure paths that notify via Logic Apps/email/Teams so issues surface immediately. Fourth, **investigate the root cause** of the intermittent failure (e.g., a source not ready at 3 AM → use a **dependency/event trigger** or a **Until/sensor** to wait for readiness). Together these eliminate the manual rerun."

### Scenario: "You need to load 500 tables from a source with one pipeline. How?"

**Answer:**
"A **metadata-driven** pipeline. I'd store table configs (name, schema, watermark column, target path) in a **control table**, use a **Lookup** to read it, then **ForEach** over the list running a **parameterized** Copy/transform per table. This avoids 500 hardcoded pipelines. I'd set a sensible **ForEach batch count** for parallelism, add **per-table error logging** so one failure doesn't stop the rest, and track each table's **watermark** for incremental loads."

### Scenario: "How do you load only new/changed records from an on-prem SQL Server into ADLS daily?"

**Answer:**
"For on-prem I need a **Self-hosted Integration Runtime** as the secure gateway. Then the **watermark pattern**: a **Lookup** reads the last-loaded `LastModifiedDate` from a control table; a **Copy activity** queries the source filtered to `> watermark`; after success, I **update the watermark** to the new max. The deltas land in **Bronze** and are **MERGED** into Silver. If the source supports it, **CDC/Change Tracking** is even better because it also captures **deletes**."

---

## 6. MongoDB / Elasticsearch / Azure SQL Scenarios

### Scenario: "You need to keep MongoDB and Elasticsearch in sync so search results are fresh. How?"

**Answer:**
"**MongoDB is the system of record; Elasticsearch is the search index.** To sync, I'd use **MongoDB Change Streams (CDC)** to capture inserts/updates/deletes and propagate them to Elasticsearch in near-real-time, rather than periodic full reindexing. I'd design **idempotent upserts** into ES (using the Mongo document `_id` as the ES doc id) so replays don't duplicate, and handle **deletes** explicitly. For a full rebuild, a batch reindex job. I'd monitor **sync lag** and have a reconciliation job to catch drift."

### Scenario: "Search queries on Elasticsearch are slow. What do you look at?"

**Answer:**
"I'd check **mappings** (are fields the right type — `keyword` vs `text`; am I analyzing fields that don't need it?), **query structure** (use **filter context** for yes/no conditions since it's cacheable and skips scoring), **shard count/size** (too many small shards or oversized shards hurt), and whether queries use expensive **wildcard/regex** or deep pagination. Often the fix is **better mappings, filters instead of queries, and right-sized shards**."

### Scenario: "When would you store data in Azure SQL vs MongoDB vs Elasticsearch?"

**Answer:**
"**Azure SQL** for **structured, relational, transactional** data needing joins, constraints, and strong consistency (master/reference data). **MongoDB** for **flexible, semi-structured documents** with evolving schemas and high document read/write. **Elasticsearch** for **full-text search and log/analytics** workloads. It's about matching the **data shape and access pattern** to the store — and sometimes using them together (Mongo + ES)."

---

# PART B — GENERAL DATA ENGINEERING SCENARIOS

## 7. Pipeline Failure & Recovery

### Scenario: "A pipeline failed overnight while writing to a Delta table. How do you recover and ensure no data corruption?"

**Theory:** Delta's **transaction log + atomic commits** mean a failed write **doesn't commit** — readers never see partial data. This is the key advantage to mention.

**Answer:**
"Because Delta writes are **atomic**, a failed mid-write **doesn't corrupt** the table — the partial files aren't committed and aren't visible. To recover: (1) check the failure in **logs/Spark UI** for root cause; (2) verify the table state with **`DESCRIBE HISTORY`** — if a bad partial logical state exists I can **time-travel/`RESTORE`** to the last good version; (3) because my write is an **idempotent `MERGE`**, I simply **re-run** the job — it reprocesses from the last **checkpoint/watermark** without duplicating. (4) Then I'd add/verify **retries and alerting** so it self-heals next time. The combination of **atomic commits + idempotency + checkpoints** is what makes recovery safe."

### Scenario: "Halfway through a batch load the job crashed. How do you avoid partial/duplicate data?"

**Answer:**
"Design for it up front: write with **`MERGE` on a key** or **overwrite the target partition** rather than blind append, so a rerun is idempotent. Use **checkpoints/watermarks** to resume from where it stopped. With Delta, the atomic commit means an incomplete write never lands. So recovery is just **re-trigger the job** — no manual cleanup."

---

## 8. Incremental Load & CDC

### Scenario: "A source table has 500M rows. You only want new/changed rows each night. Design it."

**Answer:**
"**Watermark pattern** if there's a reliable `last_modified`/incrementing id: store the last-loaded value in a control/Delta table, extract only `> watermark`, **MERGE** into the target, update the watermark. **CDC** (Debezium / DB-native change tracking) if I need **near-real-time and deletes captured** — it reads the transaction log with minimal source impact. **Trade-off:** watermark is simple but misses hard deletes and requires a trustworthy timestamp; CDC is more complete but more infrastructure. For 500M rows, incremental is essential — a full nightly reload would be far too slow and costly."

### Scenario: "The source has no modified-date column. How do you detect changes?"

**Answer:**
"Options in order of preference: (1) enable **CDC/Change Tracking** at the source if possible; (2) **hash-based diff** — compute a row hash and compare against the previous snapshot to find changed rows, then MERGE only those; (3) as a last resort, **full extract into staging + MERGE** so at least the target only changes where data actually differs. I'd push to add a reliable change-tracking mechanism at the source via a **data contract**."

---

## 9. Data Skew & Shuffle

### Scenario: "One task in your Spark stage takes 10x longer than the others. What's happening and how do you fix it?"

**Answer:**
"That's classic **data skew** — one partition has far more data because a key value is disproportionately common (e.g., a `null` or a 'default' customer). Fixes: (1) **salting** — append a random suffix to the hot key to spread it across partitions, then aggregate in two steps; (2) **broadcast** the other side if it's small to avoid the shuffle; (3) enable **AQE skew-join handling**, which splits oversized partitions at runtime; (4) **filter/handle** the skew value separately (e.g., process nulls apart). I'd confirm via the Spark UI that task durations even out."

---

## 10. Small Files & File Size

### Scenario: "A streaming job has created millions of tiny files and queries are now crawling. Fix it."

**Answer:**
"This is the **small-file problem** — excessive metadata and per-file overhead kill read performance. Fixes: (1) run **`OPTIMIZE`** to **compact** small files into larger ones (ideally ~128MB–1GB); (2) enable **Optimized Writes + Auto Compaction** so it doesn't recur; (3) for streaming, increase the **trigger interval** so each micro-batch writes fewer, larger files; (4) revisit **partitioning** — over-partitioning on a high-cardinality column is often the cause. Then **`VACUUM`** to clean up the old small files (respecting retention)."

---

## 11. Schema Drift & Evolution

### Scenario: "Your daily JSON feed suddenly has a new field and a changed type. Your pipeline breaks. How do you handle this robustly?"

**Answer:**
"For the **additive new field**, enable **schema evolution** (`mergeSchema` / Auto Loader schema evolution) so new columns are absorbed automatically. For the **type change** (breaking), I wouldn't silently coerce — I'd **quarantine** non-conforming records to a separate table, **alert**, and handle the type explicitly once I understand the change. Long term, I'd push for a **data contract** with the producer so schema changes are versioned and communicated, not surprises. The principle: **be tolerant of additive change, explicit and safe about breaking change** — never let one bad record fail the whole load silently."

---

## 12. Data Quality & Bad Data

### Scenario: "Downstream consumers complain about bad data (nulls, duplicates) reaching reports. How do you prevent it?"

**Answer:**
"Add a **data-quality gate** between layers — checks for **nulls in required fields, duplicates, referential integrity, value ranges, row-count anomalies, and freshness**. Bad records get **quarantined** (not silently dropped) for review, and the pipeline **alerts** on threshold breaches. I'd use a framework like **Great Expectations / dbt tests / Delta Live Tables expectations** to formalize and version these checks. I'd also track **quality metrics over time** so degradation is visible. The goal is to **catch issues at Silver before they reach Gold/BI**."

### Scenario: "How would you implement data quality without failing the whole pipeline on one bad row?"

**Answer:**
"Use **expectations that route, not just fail** — e.g., DLT `expect_or_drop` to drop-and-log bad rows, or write failures to a **quarantine table** while good rows proceed. Set **thresholds** (e.g., fail only if >1% of rows are bad). This keeps the pipeline flowing while preserving visibility and the ability to reprocess quarantined records."

---

## 13. Duplicates & Idempotency

### Scenario: "After a rerun, your table has duplicate records. Why, and how do you prevent it?"

**Answer:**
"The write was a **blind append**, which isn't idempotent — rerunning re-inserts the same rows. The fix is **idempotent design**: use **`MERGE` on a business key** (update matched, insert unmatched) or **overwrite the affected partition** instead of appending. To clean up existing duplicates, use **`ROW_NUMBER() OVER (PARTITION BY key ORDER BY updated_at DESC)`** and keep `rn = 1`. Combined with **checkpoints/watermarks**, reruns become safe."

```python
from pyspark.sql.window import Window
from pyspark.sql import functions as F
w = Window.partitionBy("id").orderBy(F.col("updated_at").desc())
deduped = df.withColumn("rn", F.row_number().over(w)).filter("rn = 1").drop("rn")
```

---

## 14. Streaming / Real-Time

### Scenario: "Design a real-time pipeline that ingests events from Event Hubs/Kafka, processes them, and writes to a serving store (e.g., Cosmos DB / Delta)."

**Answer:**
"Use **Spark Structured Streaming** (or Flink): read from **Event Hubs/Kafka**, apply transformations/enrichment with **windowing and watermarks** to handle late/out-of-order events, and write to the sink with a **checkpoint location** for **exactly-once** semantics. For aggregations I'd define **tumbling/sliding windows** keyed by event time. I'd monitor **consumer lag and throughput**, size the cluster for peak, and keep a **batch/replay path (Kappa)** for reprocessing by replaying the log. Sink choice: **Delta** for analytics, **Cosmos DB** for low-latency app serving."

### Scenario: "Events arrive late and out of order. How do you handle them correctly?"

**Answer:**
"Process on **event time, not processing time**, and use a **watermark** to define how long to wait for late data before finalizing a window — late events within the watermark are still included; beyond it they're dropped or sent to a side output. For correctness on updates, **dedupe with MERGE** on an event id. This gives accurate windowed results without waiting forever."

---

## 15. Cost Optimization

### Scenario: "Your Databricks/ADF bill is too high. How do you bring it down without hurting SLAs?"

**Answer:**
"Several levers: (1) **job clusters with auto-termination** instead of always-on all-purpose clusters; (2) **spot/low-priority VMs** for non-critical workloads; (3) **right-size** clusters and enable **autoscaling**; (4) **incremental loads** instead of full reloads (process less data); (5) **columnar Delta/Parquet + partition pruning + Z-Order** so queries scan less; (6) **OPTIMIZE** to fix small files; (7) **minimize shuffles** and cache wisely; (8) **serverless SQL** for spiky ad-hoc; (9) **lifecycle tiering** of cold data to cool/archive. I'd use the **Spark UI / cost dashboards** to find the most expensive jobs first and optimize those — 80/20."

---

## 16. Security & Governance

### Scenario: "How do you secure a pipeline handling PII end-to-end?"

**Answer:**
"**Identity:** authenticate services with **managed identities** (no stored credentials) via **Entra ID**; **secrets** in **Key Vault** (never hardcoded). **Access:** RBAC + **Unity Catalog** fine-grained grants, with **row-level security and column masking** on PII. **Data protection:** encryption at rest (default) and in transit (TLS); **mask/tokenize** PII as early as possible (even at Bronze if required). **Network:** Private Link/VNet. **Governance:** classify PII, track **lineage**, and keep **audit logs** for compliance (GDPR). Principle: **least privilege + encrypt + mask + audit**."

---

## 17. CI/CD & Deployment

### Scenario: "Set up CI/CD so changes to notebooks/pipelines deploy safely across dev/test/prod."

**Answer:**
"Store everything in **Git** (Databricks Repos / Azure DevOps). **CI** runs on pull requests: lint, unit tests, and **data-validation tests** on a sample. **CD** promotes through environments with **parameterized configs** (Key Vault, paths per env). For **ADF**: Git-integrated factory → **publish to ARM templates** → release pipeline deploys with environment parameters. For **Databricks**: **Asset Bundles (DABs)** or the CLI/REST API to deploy notebooks and jobs. I'd require **PR review**, keep **prod credentials isolated**, and have a **rollback** plan (redeploy previous version). This prevents 'it worked in dev' surprises."

---

## 18. System Design Scenarios

### Scenario: "Design a data platform to process millions of daily transactions for analytics and BI."

**Answer (use the framework):**
"**Requirements:** ~X M transactions/day, daily SLA (or near-real-time?), consumers = BI + analysts. **Ingest:** CDC or batch from the OLTP source into **Bronze** (raw Delta, partitioned by date). **Transform:** Spark to **Silver** (clean, dedupe, schema-enforce, **MERGE** for CDC) → **Gold** (star schema, **SCD2** dims, aggregates, **Z-Ordered**). **Serve:** Gold → warehouse/Fabric → **Power BI** (Direct Lake). **Orchestrate:** Airflow/ADF/Workflows with retries + alerts on **job clusters**. **Cross-cutting:** idempotency, data-quality gates, observability/lineage, Key Vault security, cost controls (incremental, pruning, autoscale). I'd call out the **latency vs cost** trade-off and design for **reprocessing from Bronze**."

### Scenario: "Migrate an on-prem data warehouse to the cloud. How do you plan it?"

**Answer:**
"**Assess** current schemas, volumes, and dependencies. **Connect** on-prem via **Self-hosted IR / gateway**. **Lift data** in waves into ADLS (staged as Parquet), validating with **reconciliation counts** per table. **Re-model/transform** into the lakehouse (medallion) — modernizing rather than blindly copying. **Re-point consumers** (BI) incrementally, running **parallel** old-vs-new until validated. **Cut over** table-by-table with rollback ability. Throughout: **data quality, lineage, security, and stakeholder communication**. Phased and validated, never big-bang."

---

## 19. Behavioral Scenarios (STAR)

Scenario-style behavioral questions — answer with **Situation, Task, Action, Result**.

### "Tell me about a time a pipeline broke in production. What did you do?"

**Answer (STAR):**
"*(S)* A nightly medallion pipeline failed mid-load and a downstream report was due that morning. *(T)* I needed to restore correct data fast without duplicating. *(A)* I checked logs and `DESCRIBE HISTORY`, confirmed Delta's atomic commit meant no partial corruption, and **re-triggered the idempotent MERGE job** which resumed from the watermark. I then root-caused the failure (a source file arrived late) and added an **event-based dependency + retry** so it couldn't recur. *(R)* Data was restored before the report deadline and the manual reruns stopped."

### "Tell me about a time you improved a slow process."

**Answer (STAR):**
"*(S)* Spark jobs in our pipeline were running long and inflating DBU cost. *(T)* Speed them up without changing outputs. *(A)* Diagnosed shuffles/skew/small-files in the Spark UI; applied broadcast joins, OPTIMIZE/Z-Order, repartitioning, AQE, Photon, and caching. *(R)* Cut average runtime ~30%, improving SLA reliability and reducing cost. I documented the pattern for the team."

### "A stakeholder demanded a change you thought was risky. How did you handle it?"

**Answer (STAR):**
"*(S)* A stakeholder wanted to skip data-quality checks to hit a deadline. *(T)* Balance speed vs reliability. *(A)* I explained the risk of bad data reaching reports, proposed a **compromise** — ship on time but route failures to a **quarantine table** with alerting instead of removing checks. *(R)* We met the deadline and caught two genuine data issues that week, which validated keeping the checks."

---

## 20. Rapid-Fire Scenario Drills

Quick "what would you do" prompts — practice a 2-sentence answer for each:

- **Pipeline ran twice, data duplicated** → MERGE/partition-overwrite for idempotency; dedupe with ROW_NUMBER.
- **Job slow from shuffles** → broadcast small tables, reduce shuffled data, tune partitions, AQE.
- **One task 10x slower** → data skew; salt the key / AQE skew join / broadcast.
- **Millions of tiny files** → OPTIMIZE + auto-compaction; fewer, larger writes.
- **New column in source feed** → schema evolution (mergeSchema); quarantine breaking changes.
- **Need only changed rows** → watermark or CDC; MERGE deltas.
- **On-prem source** → Self-hosted Integration Runtime.
- **500 tables, one pipeline** → metadata-driven Lookup + ForEach.
- **Failed write to Delta** → atomic commit = no corruption; re-run idempotent job; RESTORE if needed.
- **Late/out-of-order events** → event-time + watermarks + windowing.
- **Bill too high** → job clusters + auto-terminate, spot VMs, incremental, pruning, OPTIMIZE.
- **PII handling** → managed identity, Key Vault, masking, row/column security, audit.
- **OOM error** → avoid collect(), fix skew/partitions, right-size executors.
- **Wrong BI metric** → trace lineage, check grain/join fan-out, freshness, backfill.
- **Sync Mongo → Elasticsearch** → change streams (CDC), idempotent upsert by id.
- **Deploy safely across envs** → Git + CI tests + parameterized CD + rollback.

---

## 21. Last-Minute Checklist

The hour before:

- [ ] Memorize the **answer framework**: Clarify → Diagnose → Propose → Trade-offs → Productionize.
- [ ] Be able to tell the **Talend→Databricks migration** story as a scenario (phased + parallel-run validation).
- [ ] **Pipeline failure recovery**: atomic Delta commit + idempotent MERGE + checkpoints (the #1 scenario).
- [ ] **Slow job diagnosis**: Spark UI → shuffle/skew/small-files → broadcast/OPTIMIZE/AQE/Photon.
- [ ] **Incremental load**: watermark vs CDC, and the no-timestamp fallback (hash diff).
- [ ] **Data skew**: salting / broadcast / AQE.
- [ ] **Small files**: OPTIMIZE + auto-compaction.
- [ ] **Schema drift**: evolution for additive, quarantine for breaking, data contracts.
- [ ] **Idempotency**: why append duplicates, how MERGE fixes it.
- [ ] **Streaming**: event-time, watermarks, checkpoints, exactly-once.
- [ ] **Cost**: job clusters, spot, incremental, pruning, OPTIMIZE.
- [ ] **Security/PII**: managed identity, Key Vault, masking, RLS/CLS.
- [ ] **CI/CD**: Git + tests + parameterized deploy + rollback.
- [ ] Prep **3 STAR stories** (failure recovery, performance win, a disagreement).
- [ ] Practice **2 full system designs** out loud using the §18 framework.

**Interview tips:** Scenario questions reward **structured thinking over recall**. Always **clarify assumptions first**, reason through **trade-offs**, and close with **production concerns** (idempotency, monitoring, data quality, cost, security). Anchor answers to **real things you've built** — your migration, medallion, and optimization work — because applied experience beats theory every time.

---

*Good luck, Tejas — for any scenario: clarify, diagnose, propose, weigh trade-offs, and productionize. Tie it to your real projects and you'll sound exactly like the senior engineer they want to hire.*
