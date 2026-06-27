# Interview Prep — Tailored to Tejas Sakpal's Resume

> **Purpose:** Every question here is mapped to *your* resume — your projects, your skills, your numbers. For each, you'll see **what the interviewer is really probing** and **how to answer**. Prep your real examples against the project section; revise the concept banks for the theory.
>
> **Profile snapshot:** Data Engineer, ~4 yrs · Azure stack (Databricks, ADF, ADLS, Microsoft Fabric, Azure SQL) · Python / SQL / PySpark · ETL/ELT, Medallion + Delta Lake · MongoDB, Elasticsearch · Airflow, Docker, Git, CI/CD · GCP/BigQuery · Certs: Databricks DE Associate, GCP Data Engineer, GCP Database Engineer, AI‑900.

---

## Table of Contents

1. [How Your Interview Will Be Structured](#1-how-your-interview-will-be-structured)
2. [Project & Experience Deep-Dive (Resume Bullets → Questions)](#2-project--experience-deep-dive)
3. [The Numbers You Must Defend (30%, migration, etc.)](#3-the-numbers-you-must-defend)
4. [Concept-Wise Question Banks (per skill)](#4-concept-wise-question-banks)
5. [Scenario & System-Design Questions](#5-scenario--system-design-questions)
6. [Behavioral / HR Questions](#6-behavioral--hr-questions)
7. [Likely Resume Red-Flags & How to Handle Them](#7-likely-resume-red-flags--how-to-handle-them)
8. [Smart Questions to Ask the Interviewer](#8-smart-questions-to-ask-the-interviewer)
9. [Final Prep Checklist](#9-final-prep-checklist)

---

## 1. How Your Interview Will Be Structured

For a 4-year Azure Data Engineer, expect **4–5 rounds**:

1. **Screening (recruiter/HM):** resume walkthrough, stack confirmation, notice period, expectations.
2. **Technical Round 1 — Core:** SQL queries (live), Python/PySpark coding, Spark concepts.
3. **Technical Round 2 — Platform & Project:** Databricks/Delta, ADF, medallion, deep dive into *your* projects.
4. **Scenario / System Design:** "Design a pipeline to…", incremental loads, optimization, failure handling.
5. **Managerial / Behavioral:** ownership, conflict, prioritization, communication.

**Golden rule for experienced candidates:** interviewers weigh **real project examples over textbook answers**. Use the **STAR** format (Situation, Task, Action, Result) and always tie back to **what you actually built**.

---

## 2. Project & Experience Deep-Dive

This is the heart of *your* interview. Each resume bullet is a launchpad — prep a crisp STAR story for each.

### 2.1 Accenture — "Migrated legacy ETL from Talend to Azure Databricks using PySpark"

**Questions you'll get:**

- Walk me through the **migration end-to-end**. What was the source architecture, and what did the target look like?
- **Why** migrate from Talend to Databricks? What were the business/technical drivers?
- How did you ensure the migrated pipelines produced **identical results** (data parity/validation)?
- What was your **migration strategy** — big bang or phased? How did you handle rollback?
- What was the **hardest Talend component** to translate into PySpark, and how did you solve it?
- How did you handle **Talend-specific logic** (tMap, joins, lookups) in Spark?
- How did you **test** and **reconcile** old vs new outputs?

**What they're probing:** Can you own a complex migration, think about risk/validation, and translate concepts across tools — not just write Spark.

**How to answer (template):** "The legacy platform ran *N* Talend jobs doing batch ETL into *target*. We moved to a Databricks medallion design. I took a **phased approach** — migrated job-by-job, ran **parallel runs** comparing row counts, checksums, and key aggregates between Talend and Databricks outputs, and only cut over once reconciliation passed. The trickiest part was *<a real example — e.g., a tMap with complex lookups>*, which I reimplemented as **broadcast joins + window functions** in PySpark." Have **one concrete component** ready to describe in detail.

### 2.2 Accenture — "Built end-to-end pipelines with ADF + Databricks + ADLS, medallion on Delta Lake"

**Questions:**

- Draw your **medallion architecture**. What lands in Bronze vs Silver vs Gold?
- What **transformations** happen at each layer? Where do you dedupe / enforce schema / apply business logic?
- How does **ADF orchestrate** Databricks here — what's ADF's job vs Databricks' job?
- How do you handle **incremental loads** into Bronze? (watermark? Auto Loader?)
- How do you implement **SCD Type 1 / Type 2** in Silver/Gold using Delta `MERGE`?
- How do you ensure **idempotency** so re-runs don't duplicate data?
- What's your **partitioning** strategy on the Delta tables?

**What they're probing:** Do you understand *why* the medallion pattern exists and can you defend your design choices?

**How to answer:** Bronze = raw, append-only, source-faithful (for reprocessing/audit). Silver = cleaned, deduped, schema-enforced, conformed/joined. Gold = business aggregates/marts for BI/ML. ADF orchestrates (triggers, dependencies, scheduling, parameter passing); Databricks does the heavy transforms. Mention **`MERGE` for upserts**, **watermark or Auto Loader** for incrementals, and **checkpoints/idempotent writes**.

### 2.3 Accenture — "Optimized Spark jobs… 30% increase in processing efficiency"

**This is your headline number — they WILL drill it.**

- **What exactly** did you optimize? Walk me through before vs after.
- How did you **measure** the 30%? (runtime? cost/DBU? throughput?)
- How did you **diagnose** the bottleneck — Spark UI, query plan, `explain()`?
- What was the root cause — **shuffles, skew, small files, bad partitioning, UDFs?**
- Which specific techniques moved the needle?

**How to answer:** Be specific and quantified. Example arc: "Jobs were slow due to **excessive shuffles and the small-file problem** from frequent appends. I (1) replaced a large-table join's small side with a **broadcast join**, (2) ran **OPTIMIZE + Z-ORDER** on the hot Delta tables for data skipping, (3) **repartitioned** to cut skew, (4) enabled **AQE** and **Photon**, and (5) cached a reused dimension. I measured **average job runtime** over a week before/after via the Spark UI / job metrics — it dropped ~30%, which also reduced DBU cost." Have the **diagnosis → action → measurement** chain memorized.

### 2.4 Accenture — "Orchestrated and scheduled workflows in ADF, reducing manual reruns"

**Questions:**

- How do you build **failure handling / retries** in ADF? What about **alerting**?
- How do you handle **dependencies** between activities/pipelines?
- How did you **reduce manual reruns** specifically? (idempotency? checkpointing? retry policies?)
- Triggers used — **schedule, tumbling window, event-based**? Difference?
- How do you do **parameterization** and reuse across pipelines?

**How to answer:** Mention **retry policy + timeout** on activities, **tumbling window triggers** for dependency/backfill, **failure paths** (on-failure activities) with **Logic Apps/email alerts**, and **idempotent/restartable** design so a failed run resumes cleanly instead of a full manual rerun.

### 2.5 SLO Technologies (AdvaRisk) — ADF pipelines, ADLS modeling, Python/SQL tuning, 30% automation

**Questions:**

- How did you **model data in ADLS**? Folder/zone structure, file formats, partitioning?
- What **transformation logic** did you tune in Databricks, and how did tuning improve **data quality**?
- The **30% reduction in manual intervention** — what did you automate and how?
- Working in **cross-functional teams** — how did you gather requirements and hand off to analytics?

**How to answer:** ADLS modeling = zone layout (raw/curated), **Parquet/Delta**, partition by date/business key, naming conventions. Automation = Python scripts/parameterized ADF replacing manual ingestion + scheduling + validation checks. (Note: this is a **risk/fintech** domain — be ready to discuss **data quality and reliability** seriously.)

### 2.6 Innoplexus — MongoDB, Elasticsearch, Azure SQL, SDLC/CI-CD

**Questions:**

- Why **MongoDB** (document store) and **Elasticsearch** (search) for that use case — what data and access patterns?
- How did you **model documents** vs index for search? Any **denormalization** trade-offs?
- How did you load/transform data into **Elasticsearch** — mappings, analyzers?
- **Azure SQL** vs the NoSQL stores — when relational vs document/search?
- Your role in **CI/CD** — what got automated, what tools?

**What they're probing:** Breadth beyond Spark — NoSQL/search reasoning and SDLC maturity.

**How to answer:** Document store for flexible/semi-structured data and fast key/document reads; Elasticsearch for **full-text search and analytics** with inverted indexes/mappings; Azure SQL for **structured, relational, transactional** data needing joins/constraints. Mention **Git + pipelines (CI/CD)** and testing in releases.

---

## 3. The Numbers You Must Defend

Interviewers love attacking quantified claims. Prep these three:

| Claim | Be ready to explain |
|---|---|
| **30% processing efficiency (Spark)** | What metric, how measured, which techniques (broadcast, OPTIMIZE/Z-Order, AQE, Photon, repartition, cache), before/after baseline. |
| **30% reduced manual intervention (Python automation)** | What was manual before, what you scripted/scheduled, how you quantified hours/effort saved. |
| **Talend → Databricks migration** | Scope (# pipelines), strategy (phased), validation (parallel runs/reconciliation), timeline, rollback. |

**Tip:** If you don't have exact figures, give an honest **estimation method** ("measured average job runtime across the daily batch over two weeks"). Vague numbers you can't defend hurt more than they help.

---

## 4. Concept-Wise Question Banks

Quick-fire theory by skill. (You already have deep guides for Python, SQL, PySpark, and Databricks — this is the targeted shortlist most likely from *your* stack.)

### 4.1 Python (for Data Engineering)
- List vs tuple vs set vs dict; mutability; shallow vs deep copy.
- Generators & `yield` (why memory-efficient for large files).
- `*args`/`**kwargs`, decorators, context managers (`with`).
- Exception handling; reading large files in chunks.
- **Pandas:** when to use Pandas vs PySpark (single-node vs distributed); `merge`, `groupby`, vectorization.
- How do you make a Python ingestion script **idempotent and restartable**?

### 4.2 SQL
- Joins (inner/left/anti/semi), `GROUP BY` vs `HAVING`, window functions (`ROW_NUMBER`/`RANK`/`DENSE_RANK`, running totals).
- **Nth highest salary**, **find/delete duplicates**, **top-N per group** (live coding likely).
- CTEs vs subqueries; correlated subqueries.
- Indexing, query optimization, execution plans.
- `DELETE` vs `TRUNCATE` vs `DROP`; normalization (1NF/2NF/3NF) vs denormalization.

### 4.3 PySpark
- RDD vs DataFrame; lazy evaluation; transformations vs actions.
- **Narrow vs wide transformations; shuffle**; jobs/stages/tasks.
- `repartition` vs `coalesce`; broadcast joins; caching/persist.
- Handling **data skew** (salting), small-file problem, AQE.
- Why avoid UDFs; window functions; `MERGE` for upserts.
- `cache` vs `checkpoint`.

### 4.4 Azure Databricks & Delta Lake
- Control plane vs data plane; all-purpose vs job cluster; autoscaling vs auto-termination.
- Delta = Parquet + `_delta_log`; **ACID, time travel, schema evolution**.
- `OPTIMIZE`, `ZORDER`, `VACUUM`, Liquid Clustering; data skipping.
- Managed vs external tables (drop behavior).
- Unity Catalog (3-level namespace, governance, lineage); Delta Live Tables vs notebooks.
- Auto Loader (`cloudFiles`) for incremental ingestion; Structured Streaming + checkpoints.

### 4.5 Azure Data Factory (ADF)
- **Integration Runtime** types (Azure, Self-hosted, SSIS) — when each.
- **Mapping Data Flows vs Copy Activity vs calling Databricks** — when to use which.
- **Triggers:** schedule vs **tumbling window** vs event-based.
- **Incremental load / watermark pattern** (Lookup → filter on LastModifiedDate/ID → update watermark).
- Parameters & variables; **Linked Services** & **Datasets**; **ForEach/Lookup/If** activities.
- Error handling, retries, monitoring, alerting; CI/CD for ADF (ARM templates / Git).

### 4.6 ADLS Gen2 & Storage
- ADLS Gen2 vs Blob (hierarchical namespace); access via service principal/managed identity/UC external locations.
- File formats — **Parquet/Delta vs CSV/JSON** (columnar, compression, pushdown).
- Partitioning strategy; zone/folder design; lifecycle management.

### 4.7 Microsoft Fabric (you list it — expect at least one question)
- What is Fabric and how does it relate to Synapse/Power BI? **OneLake**, Lakehouse vs Warehouse, **Direct Lake** mode, Dataflows Gen2.
- Fabric vs Databricks — where do they overlap/differ?

### 4.8 Apache Airflow
- DAGs, operators, tasks, scheduling; **idempotent tasks**.
- Airflow vs ADF — when would you use each?
- XComs, sensors, retries, backfills.

### 4.9 MongoDB & Elasticsearch
- MongoDB: documents/collections, indexing, aggregation pipeline, sharding/replication, schema design.
- Elasticsearch: inverted index, mappings, analyzers, query vs filter context, near-real-time search.
- When NoSQL/search vs relational.

### 4.10 Data Warehousing & Modeling
- **Star vs snowflake schema**; fact vs dimension tables; grain.
- **SCD Type 1 vs Type 2** (implement with Delta `MERGE`).
- OLTP vs OLAP; normalization vs denormalization for analytics.
- Surrogate keys, slowly changing dimensions, late-arriving data.

### 4.11 DevOps / CI-CD / Docker / Git
- CI/CD for data pipelines (Databricks Repos/DABs, ADF ARM, GitHub Actions/Azure DevOps).
- Docker basics; why containerize; Git branching/PR workflow.

### 4.12 GCP / BigQuery (you're certified — possible cross-questions)
- BigQuery architecture (serverless, columnar, slots); partitioning vs clustering.
- BigQuery vs Synapse/Databricks SQL; cost control (avoid `SELECT *`, partition pruning).
- Dataflow/Dataproc/Composer at a high level.

### 4.13 Generative AI (listed — be ready for a light question)
- How have you used GenAI in your work (code assist, doc/SQL generation, RAG)?
- RAG basics; embeddings + vector search (ties to your Elasticsearch experience).

---

## 5. Scenario & System-Design Questions

These are **highly likely** for 4-year DEs. Practice answering out loud.

1. **Design an end-to-end pipeline** to ingest daily files from a vendor into ADLS, transform via medallion in Databricks, and serve to Power BI. Walk through ADF orchestration, incremental loads, and failure handling.
2. **Incremental load:** A source table has millions of rows; you only want new/changed records each night. How? *(Answer: watermark column via ADF Lookup, or CDC, or Auto Loader for files.)*
3. **Large fact × dimension join is slow.** How do you optimize? *(Broadcast small dims, partition/bucket the fact on join key, cache dims, OPTIMIZE/Z-Order, AQE.)*
4. **Late-arriving / out-of-order data** in streaming — how do you handle it? *(Watermarks, dedupe with `MERGE`, idempotent writes.)*
5. **Data quality framework** — how do you enforce it across Bronze→Gold? *(Schema enforcement, DLT expectations, null/dup checks, reconciliation counts.)*
6. **A nightly job failed at 3 AM and produced partial data.** How does your design avoid corruption? *(Atomic Delta commits, idempotency, checkpoints, retries, alerting.)*
7. **Handle schema drift** in incoming files. *(Auto Loader schema evolution, `mergeSchema`, quarantine bad records.)*
8. **Cost optimization** on Databricks/ADF. *(Job clusters + auto-termination, spot instances, right-sizing, Photon, fewer shuffles, Parquet/Delta.)*
9. **Migrate a Databricks workspace across regions** — what's your plan? *(Back up data/notebooks, export configs, recreate workspace, restore, repoint storage.)*
10. **Data skew** is overloading a few tasks. Diagnose and fix. *(Salting, AQE skew join, broadcast.)*

**Framework to answer any design question:** Requirements/SLA → Source & ingestion (batch/stream, incremental) → Storage/zones (medallion) → Transform (Spark/Delta) → Orchestration (ADF/Airflow) → Quality & monitoring → Serving (warehouse/BI) → Cost & security.

---

## 6. Behavioral / HR Questions

Use **STAR**. Prep one real story for each:

- Tell me about a **challenging pipeline/bug** you solved. *(→ use the migration or the Spark optimization.)*
- A time you **improved performance or reliability**. *(→ the 30% story.)*
- A time you **disagreed** with a teammate/architect on a design.
- How do you **prioritize** when multiple pipelines break at once?
- A time you **missed a deadline or made a mistake** — what did you learn?
- How do you keep data **reliable/high-quality** under pressure? *(fintech/risk context at AdvaRisk.)*
- Why are you **leaving / looking** to change? Where do you see yourself in 3 years?
- How do you stay **current** (certs show this — mention Databricks/GCP/Azure).

---

## 7. Likely Resume Red-Flags & How to Handle Them

Be ready — interviewers probe gaps and inconsistencies:

- **Two "30%" improvements** — make sure each has a distinct, defensible measurement method. Don't let them sound copy-pasted.
- **Broad stack (Azure + GCP + Mongo + ES + Fabric + Airflow + GenAI)** — they may test depth on something you listed but used lightly. **Be honest about depth:** "Production depth in Azure/Databricks/PySpark; familiar with GCP/BigQuery via certification and POCs." Never bluff a tool you barely touched.
- **Microsoft Fabric & Generative AI** look like newer/lighter skills — have a 2–3 sentence honest answer for each ready.
- **Short-ish tenures** (3 roles in ~3 years before Accenture) — frame as growth and increasing responsibility, not instability.
- **Certs vs hands-on** — for GCP especially, expect "have you used this in production or just certified?" Answer truthfully.

**Principle:** Depth on your **core** (Azure/Databricks/PySpark/SQL) wins. For peripheral skills, show **conceptual understanding + willingness to learn** rather than overclaiming.

---

## 8. Smart Questions to Ask the Interviewer

Asking good questions signals seniority:

- What does the current **data platform/architecture** look like, and what's the biggest pain point?
- Are you on **Unity Catalog / Delta Live Tables / Fabric** yet, or migrating toward them?
- How is the team split between **data engineering, analytics, and platform**?
- What does **success in this role** look like in the first 6 months?
- How do you handle **data quality, governance, and cost** at scale?
- What's the **CI/CD and testing** maturity for pipelines here?

---

## 9. Final Prep Checklist

The day before:

- [ ] Rehearse a **2-minute resume walkthrough** (story arc: Innoplexus → SLO → Accenture, increasing scope).
- [ ] Prep **STAR stories**: Talend→Databricks migration, Spark 30% optimization, automation 30%, a data-quality save, a disagreement.
- [ ] Be able to **draw your medallion architecture** on a whiteboard from memory.
- [ ] Memorize your **optimization toolkit**: broadcast, OPTIMIZE/Z-Order, partitioning, AQE, Photon, cache, repartition.
- [ ] Drill **live SQL**: 2nd-highest salary, dedupe, top-N per group, running total.
- [ ] Drill **live PySpark**: read→transform→write Delta, `MERGE` upsert, window functions, word count.
- [ ] Review **ADF incremental/watermark pattern** and trigger types.
- [ ] Prepare honest **depth statements** for GCP, Fabric, GenAI, Mongo/ES.
- [ ] Prepare **3 questions** to ask them.
- [ ] Revise your four concept guides (Python, SQL, PySpark, Databricks).

**Mindset:** You have a strong, coherent Azure DE profile with a clean migration + optimization narrative and solid certs. Lead with **real project stories**, quantify honestly, and go deep on your core stack. That combination is exactly what interviewers want from a 4-year data engineer.

---

*Good luck, Tejas. Your story is strong — own the projects, defend the numbers, and stay honest about depth. That wins interviews.*
