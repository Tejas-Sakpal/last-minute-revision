# Tejas Sakpal — Resume-Based Interview Q&A (With Full Answers)

> **What this is:** Every likely interview question mapped to *your* resume — **project-related, concept-wise, and behavioral** — each with a **complete, ready-to-say answer**. The project answers use placeholders like *<N pipelines>* where you should drop in your real numbers. Read the answer, then make it yours.
>
> **Profile:** Data Engineer, ~4 yrs · Azure (Databricks, ADF, ADLS, Microsoft Fabric, Azure SQL) · Python / SQL / PySpark · ETL/ELT, Medallion + Delta Lake · MongoDB, Elasticsearch · Airflow, Docker, Git, CI/CD · GCP/BigQuery · Certs: Databricks DE Associate, GCP Data Engineer, GCP Database Engineer, AI‑900.

---

## Table of Contents

1. [Resume Walkthrough & "Tell Me About Yourself"](#1-resume-walkthrough--tell-me-about-yourself)
2. [Project Deep-Dive — Accenture (Talend→Databricks, Medallion, Spark, ADF)](#2-project-deep-dive--accenture)
3. [Project Deep-Dive — SLO Technologies / AdvaRisk](#3-project-deep-dive--slo-technologies--advarisk)
4. [Project Deep-Dive — Innoplexus (MongoDB, Elasticsearch, Azure SQL)](#4-project-deep-dive--innoplexus)
5. [Defending Your Numbers (the two "30%" + migration)](#5-defending-your-numbers)
6. [Concept Q&A — Python](#6-concept-qa--python)
7. [Concept Q&A — SQL](#7-concept-qa--sql)
8. [Concept Q&A — PySpark](#8-concept-qa--pyspark)
9. [Concept Q&A — Azure Databricks & Delta Lake](#9-concept-qa--azure-databricks--delta-lake)
10. [Concept Q&A — Azure Data Factory](#10-concept-qa--azure-data-factory)
11. [Concept Q&A — ADLS, Fabric, Synapse](#11-concept-qa--adls-fabric-synapse)
12. [Concept Q&A — MongoDB & Elasticsearch](#12-concept-qa--mongodb--elasticsearch)
13. [Concept Q&A — Airflow, Docker, CI/CD, GCP/BigQuery](#13-concept-qa--airflow-docker-cicd-gcpbigquery)
14. [Data Modeling & Warehousing](#14-data-modeling--warehousing)
15. [Scenario / Design Questions (with answers)](#15-scenario--design-questions)
16. [Behavioral Questions (with answers)](#16-behavioral-questions)
17. [Honest "Depth" Answers for Lighter Skills](#17-honest-depth-answers-for-lighter-skills)
18. [Questions to Ask the Interviewer](#18-questions-to-ask-the-interviewer)
19. [Final Checklist](#19-final-checklist)

---

## 1. Resume Walkthrough & "Tell Me About Yourself"

**Q: Tell me about yourself / walk me through your resume.**

**Answer:**
"I'm a data engineer with around four years of experience building and optimizing data pipelines on the Azure stack. I started at **Innoplexus** as an Associate Software Engineer, where I built ingestion and integration pipelines into **MongoDB and Elasticsearch** and worked with **Azure SQL**, which gave me strong fundamentals in data integration and the full SDLC including CI/CD. I then moved to **SLO Technologies (AdvaRisk)**, a risk-analytics product, as a Junior Data Engineer — there I designed **Azure Data Factory** pipelines, modeled data in **ADLS**, and tuned Python/SQL transformation logic in **Databricks**, while automating ingestion to cut manual effort by about 30%. Currently at **Accenture**, I lead work **migrating legacy Talend ETL to Azure Databricks with PySpark**, building end-to-end pipelines on a **medallion architecture with Delta Lake**, and I optimized Spark jobs to improve processing efficiency by about 30%. Along the way I earned the **Databricks Data Engineer Associate** and two **Google Cloud** certifications. My core strength is the Azure data engineering stack — Databricks, ADF, PySpark, and Delta — with a focus on reliable, well-modeled, analytics-ready data."

**Tip:** Keep it ~90 seconds. The arc is *growing scope*: integration → pipeline design → migration + optimization leadership.

---

## 2. Project Deep-Dive — Accenture

*(Packaged App Development Analyst, Jan 2025–Present)*

### Talend → Databricks Migration

**Q: Walk me through the Talend-to-Databricks migration. Why was it done?**

**Answer:**
"The legacy platform ran ETL on **Talend**, which was becoming costly to license and hard to scale for growing data volumes. The goal was to **modernize onto Azure Databricks** for scalable, distributed processing and a unified lakehouse. There's no one-click converter, so we **audited each Talend job** and **re-implemented its logic in PySpark/Spark SQL** inside Databricks notebooks. Talend's row-by-row, component-based flow (tMap, lookups, joins) had to be rethought as **set-based, distributed Spark transformations**. We landed the output on a **medallion architecture on Delta Lake** and orchestrated it with **ADF**. I took a **phased approach** — migrating job-by-job and running **parallel runs** to reconcile old vs new outputs before cut-over."

**Q: How did you ensure the migrated pipeline produced identical results?**

**Answer:**
"I validated with **parallel runs**: for each migrated job I ran Talend and Databricks side-by-side on the same input and compared **row counts, key aggregates (sums/averages on critical columns), and record-level checksums/hashes** on a primary key. I also checked **schema and null distributions**. Only when reconciliation matched within tolerance did we decommission the Talend job. Any mismatch was investigated — usually a difference in **null handling, date formats, or implicit type casting** between the two engines."

**Q: What was the hardest part of the migration?**

**Answer:**
"Translating **Talend's tMap with complex lookups and conditional logic** into Spark. In Talend it's row-based; in Spark I had to express it as **joins plus `when/otherwise` logic and window functions**, and be careful about **broadcast joins** for the small lookup tables to avoid shuffles. The other challenge was **orchestration parity** — Talend's job scheduling moved to **ADF triggers and dependencies**, so I rebuilt the dependency chains and added retries/alerts."

### Medallion + Delta Pipeline

**Q: Explain the medallion architecture you built.**

**Answer:**
"Data lands in **Bronze** as raw, append-only Delta — an exact copy of the source for auditability and reprocessing. **Silver** is where I **deduplicate, enforce schema, cast types, handle nulls, and join/conform** the data into a trusted view. **Gold** holds **business-level aggregates and marts** optimized for reporting and analytics. **ADF orchestrates** the flow — triggering Databricks notebooks per layer — and each Delta write uses **`MERGE`** for idempotent upserts so re-runs don't duplicate data. The big advantage is that if business logic changes, I can rebuild Silver and Gold from untouched Bronze."

**Q: How do you handle incremental loads into Bronze?**

**Answer:**
"Two patterns depending on source. For **database sources** via ADF, I use the **watermark pattern** — store the last-loaded `LastModifiedDate`/ID in a control table, Lookup it, copy only newer rows, then update the watermark. For **file sources** in Databricks, I use **Auto Loader (`cloudFiles`)**, which incrementally picks up only new files with checkpointing and schema evolution. Both feed an idempotent **`MERGE`** so the layer stays consistent on reruns."

### Spark Optimization (30%)

**Q: What exactly did you optimize to get the 30% improvement?**

**Answer:**
"I started by **diagnosing in the Spark UI** — looking at stage times, shuffle read/write, and skew, and reading the physical plan with `explain()`. The bottlenecks were **excessive shuffles, data skew on a join key, and the small-file problem** from frequent appends. My fixes: (1) converted a large-with-small join to a **broadcast join**; (2) ran **`OPTIMIZE` + `ZORDER`** on hot Delta tables for **data skipping**; (3) **repartitioned** to reduce skew and right-sized partition counts; (4) enabled **AQE** and **Photon**; and (5) **cached** a reused dimension. I measured **average job runtime across the daily batch over about two weeks** before and after — it dropped roughly 30%, which also reduced DBU cost."

**Q: How did you handle data skew specifically?**

**Answer:**
"For the skewed join I used **salting** — adding a random prefix to the hot key to spread it across more partitions — and enabled **AQE skew-join handling**, which splits large partitions at runtime. Where the small side fit in memory, **broadcasting** it avoided the shuffle entirely."

### ADF Orchestration

**Q: How did you reduce manual reruns with ADF?**

**Answer:**
"Three things. First, **retry policies and timeouts** on activities so transient failures self-heal. Second, **idempotent, restartable design** — watermarks and `MERGE` mean a failed run can simply be re-triggered without duplicating or corrupting data. Third, **on-failure paths with alerting** via Logic Apps/email, so issues surface immediately instead of being discovered the next morning. I also used **tumbling window triggers** where I needed dependency handling and backfill. Together these sharply cut the manual reruns the team used to do."

---

## 3. Project Deep-Dive — SLO Technologies / AdvaRisk

*(Jr. Data Engineer, Sep 2023–Jan 2025 — risk analytics)*

**Q: How did you model data in ADLS to support analytics?**

**Answer:**
"I organized the lake into **zones** — raw/landing, cleansed, and curated — and stored analytics data in **Parquet/Delta** for columnar compression and predicate pushdown. I **partitioned** large datasets by date and business keys for efficient pruning, kept **consistent naming conventions**, and applied **lifecycle policies** to tier older data. This gave downstream teams fast, predictable retrieval and kept storage costs down."

**Q: What transformation logic did you tune, and how did it improve data quality?**

**Answer:**
"I tuned **Python and SQL transformations in Databricks** — refactoring inefficient row-wise logic into **set-based Spark operations**, fixing joins that caused shuffles, and adding **explicit schema enforcement and validation** (null checks, dedup, type casting, referential checks). Better-structured logic not only ran faster but **caught bad records earlier**, which improved the reliability of the risk data feeding downstream analytics — important in a fintech/risk context where accuracy is critical."

**Q: The 30% reduction in manual intervention — what did you automate?**

**Answer:**
"Ingestion and collection were partly manual — someone kicked off scripts and checked outputs. I built **parameterized Python ingestion pipelines** with scheduling and **built-in validation/logging**, so data flowed automatically with alerts on failure. I estimated the saving by comparing the **recurring manual hours per week** before and after — roughly a 30% cut in manual effort, plus fewer errors from manual steps."

**Q: How did you work in cross-functional teams?**

**Answer:**
"I gathered requirements from analysts and product stakeholders, translated them into pipeline specs, and delivered **analytics-ready datasets** with documentation on schema and refresh cadence. I kept tickets in **Jira**, used **Git** for version control, and communicated changes/SLAs clearly so downstream teams could rely on the data."

---

## 4. Project Deep-Dive — Innoplexus

*(Associate Software Engineer, Jul 2022–Sep 2023)*

**Q: Why MongoDB and Elasticsearch for that use case?**

**Answer:**
"The data was **semi-structured and document-shaped**, and users needed **fast full-text search**. **MongoDB** was a natural fit for storing flexible documents with varying fields, and **Elasticsearch** handled the **full-text search and analytics** with its inverted index and analyzers. A common pattern is **MongoDB as the system of record and Elasticsearch as the search layer** — Mongo stores the structured documents, and we index searchable fields into Elasticsearch for low-latency queries. I built pipelines to **ingest, transform, and sync** data into both."

**Q: How did you model documents vs index for search?**

**Answer:**
"In MongoDB I designed documents around **access patterns** — embedding related data that's read together to avoid joins, and referencing where data is large or shared, plus **indexes** on frequently queried fields. For Elasticsearch I defined **mappings** (field types, analyzers) deliberately — choosing text vs keyword types, analyzers for tokenization, and only indexing fields that needed to be searchable to control index size. I was mindful of the **denormalization trade-off**: faster reads/search at the cost of keeping the two stores in sync."

**Q: When would you choose Azure SQL over MongoDB/Elasticsearch?**

**Answer:**
"**Azure SQL** when the data is **structured and relational** with strong consistency, joins, and transactional needs — for example reference/master data with constraints. **MongoDB** for flexible, evolving document schemas and high-write document workloads. **Elasticsearch** specifically for **search and log/analytics** use cases. It's about matching the store to the data shape and access pattern."

**Q: What was your role in CI/CD and SDLC?**

**Answer:**
"I participated in the full lifecycle — development with **Git** branching and pull requests, code review, automated **build/test/deploy via CI/CD pipelines**, and supporting deployments to production. I also contributed to **architecture discussions** for cloud-based data solutions, which gave me early exposure to system design."

---

## 5. Defending Your Numbers

**Q: You claim two separate 30% improvements — how did you measure each?**

**Answer:**
"They're different metrics. The **Spark 30%** is **processing efficiency** — I measured **average job runtime across the daily batch over about two weeks** before and after the optimizations (broadcast joins, OPTIMIZE/Z-Order, AQE, Photon, repartitioning, caching), which also reduced DBU cost. The **SLO 30%** is **reduced manual intervention** — I compared the **recurring manual hours per week** for ingestion/validation before and after I automated those steps with parameterized Python pipelines. Each number comes from a concrete before/after baseline, not a guess."

**Tip:** If pressed for exact figures you don't have, give the **measurement method** confidently rather than inventing precise numbers.

---

## 6. Concept Q&A — Python

**Q: List vs tuple vs set vs dict?**
"**List** — ordered, mutable, allows duplicates. **Tuple** — ordered, immutable, hashable (usable as a dict key). **Set** — unordered, unique elements, O(1) membership. **Dict** — key–value pairs, keys unique and hashable, insertion-ordered since 3.7."

**Q: What is a generator and why use it in data engineering?**
"A generator uses **`yield`** to produce values **lazily**, one at a time, instead of building a full list in memory. It's ideal for **streaming/processing large files** row-by-row without loading everything — low memory footprint."

**Q: Pandas vs PySpark — when?**
"**Pandas** for single-node, in-memory data that fits comfortably on one machine — quick analysis, small/medium files. **PySpark** for **distributed** processing of large data across a cluster. I use Pandas for prototyping or small lookups, PySpark for production-scale transforms."

**Q: How do you make a Python ingestion script idempotent?**
"Design it so re-running produces the same result — e.g., **upsert/MERGE on a key** rather than blind insert, **overwrite a target partition**, or check a **watermark/processed-files log** before loading. Combined with checkpointing, a rerun won't duplicate data."

**Q: Decorators / context managers?**
"A **decorator** wraps a function to add behavior (logging, timing, retries) without changing it. A **context manager** (`with`) guarantees setup/teardown — like closing a file or connection even on error. Both keep data code clean and safe."

---

## 7. Concept Q&A — SQL

**Q: WHERE vs HAVING?**
"**WHERE** filters individual rows **before** grouping and can't use aggregates. **HAVING** filters **groups after** `GROUP BY` and can use aggregates like `COUNT()`/`SUM()`."

**Q: Find the second-highest salary.**
"`SELECT MAX(salary) FROM emp WHERE salary < (SELECT MAX(salary) FROM emp);` — or with a window: `DENSE_RANK() OVER (ORDER BY salary DESC)` and filter rank = 2."

**Q: row_number vs rank vs dense_rank?**
"All rank within an ordered window. **row_number** gives a unique sequential number; **rank** gives ties the same rank then **skips**; **dense_rank** gives ties the same rank with **no gaps**."

**Q: How do you remove duplicates keeping the latest?**
"Use `ROW_NUMBER() OVER (PARTITION BY business_key ORDER BY updated_at DESC)` and keep `rn = 1`."

**Q: DELETE vs TRUNCATE vs DROP?**
"**DELETE** removes selected rows (with WHERE) and is logged/rollback-able. **TRUNCATE** removes all rows fast, resets identity, usually can't rollback. **DROP** removes the entire table structure and data."

**Q: How do you optimize a slow SQL query?**
"Check the **execution plan**, add **indexes** on filter/join columns, avoid `SELECT *`, filter early, avoid functions on indexed columns in WHERE, and ensure join columns are the same type. Partition large tables where supported."

---

## 8. Concept Q&A — PySpark

**Q: Transformation vs action; lazy evaluation?**
"**Transformations** (filter, select, join) are **lazy** — they build a DAG but don't run. **Actions** (count, show, write) trigger execution. Laziness lets Spark **optimize the whole chain** before running."

**Q: Narrow vs wide transformation?**
"**Narrow** (map, filter) — each input partition maps to one output partition, no network movement. **Wide** (groupBy, join, distinct) — needs data from multiple partitions, triggering a **shuffle** and a new stage. Minimizing shuffles is the main performance lever."

**Q: repartition vs coalesce?**
"**repartition(n)** does a **full shuffle** and can increase or decrease partitions evenly. **coalesce(n)** only **decreases** partitions and avoids a full shuffle by merging — cheaper, used before writing output."

**Q: How do you optimize a PySpark job?**
"Minimize shuffles, use **broadcast joins** for small tables, **cache** reused DataFrames, handle **skew** with salting, fix the **small-file problem** with OPTIMIZE, prefer **built-in functions over UDFs**, and enable **AQE/Photon**. I diagnose with the Spark UI and `explain()`."

**Q: Why avoid UDFs?**
"Python UDFs cause **serialization overhead** between the JVM and Python and are a **black box to Catalyst**, so no optimization or pushdown. I prefer built-in `F.*` functions; if a UDF is unavoidable, I use a **vectorized pandas UDF**."

**Q: cache vs checkpoint?**
"**Cache** stores data in memory/disk but **keeps lineage**. **Checkpoint** writes to reliable storage and **truncates lineage** — used to break very long DAGs and for streaming fault tolerance."

---

## 9. Concept Q&A — Azure Databricks & Delta Lake

**Q: What is Delta Lake and how is it different from Parquet?**
"Delta is **Parquet plus a transaction log (`_delta_log`)**. It adds **ACID transactions, time travel, schema enforcement/evolution, and MERGE/UPDATE/DELETE** — things plain Parquet can't do."

**Q: How does Delta provide ACID?**
"Through the transaction log and **optimistic concurrency control** — each write commits atomically, readers get a consistent **snapshot**, and conflicting writes retry."

**Q: Time travel — use case?**
"Query or restore a previous version with `VERSION AS OF`/`TIMESTAMP AS OF`. I use it to **roll back a bad load** or audit changes via `DESCRIBE HISTORY`."

**Q: OPTIMIZE, ZORDER, VACUUM?**
"**OPTIMIZE** compacts small files; **ZORDER** co-locates data on filter columns for **data skipping**; **VACUUM** deletes old unreferenced files (default 7-day retention — going lower breaks time travel)."

**Q: Managed vs external table?**
"For a **managed** table Databricks owns the data, so `DROP` deletes the files. For an **external** table you specify a LOCATION and `DROP` keeps the files."

**Q: All-purpose vs job cluster?**
"**All-purpose** for interactive notebook development, costlier. **Job cluster** is spun up per scheduled job and terminated on completion — cheaper and isolated, which I use for production."

**Q: What is Unity Catalog?**
"Databricks' **centralized governance** layer — a **three-level namespace** (`catalog.schema.table`), **fine-grained access control** down to row/column, **lineage**, and audit, replacing the legacy Hive metastore."

---

## 10. Concept Q&A — Azure Data Factory

**Q: Linked Service vs Dataset vs Activity vs Pipeline?**
"**Linked Service** = the connection (where + credentials). **Dataset** = a pointer to specific data within it. **Activity** = a single step. **Pipeline** = the workflow grouping activities. The **Integration Runtime** is the compute that runs them."

**Q: Integration Runtime types?**
"**Azure IR** for cloud sources, **Self-hosted IR** for on-prem/private networks (acts as a secure gateway), and **Azure-SSIS IR** to run legacy SSIS packages."

**Q: How do you implement incremental load in ADF?**
"The **watermark pattern**: keep the last-loaded timestamp/ID in a control table, **Lookup** it, **Copy** only rows greater than it, then **update** the watermark. For files, **Auto Loader** in Databricks."

**Q: Trigger types?**
"**Schedule** (clock-based), **tumbling window** (contiguous non-overlapping windows with backfill, dependencies, retries), and **event-based** (fires on file arrival via Event Grid)."

**Q: Copy Activity vs Mapping Data Flow vs Databricks?**
"**Copy** moves data. **Mapping Data Flow** does visual, code-free transforms on managed Spark. **Databricks notebooks** give code-first, large-scale transforms — what I use for complex logic."

**Q: How do you build a metadata-driven pipeline?**
"Parameterize datasets/linked services, store table configs in a **control table**, then **Lookup + ForEach** to loop over them — one pipeline handles many tables."

---

## 11. Concept Q&A — ADLS, Fabric, Synapse

**Q: ADLS Gen2 vs Blob?**
"ADLS Gen2 is Blob **plus a hierarchical namespace** — real directories, atomic folder operations, and POSIX ACLs, optimized for analytics."

**Q: What is Microsoft Fabric?**
"A **SaaS unified analytics platform** built on **OneLake** — a single tenant-wide data lake storing **one copy** of data in Delta/Parquet, used by all engines. It unifies Data Factory, Data Engineering, Warehouse, Data Science, Real-Time, and Power BI."

**Q: OneLake and Direct Lake?**
"**OneLake** is 'OneDrive for data' — one logical lake per org; **shortcuts** reference external data without copying. **Direct Lake** lets Power BI read Delta tables directly from OneLake, combining Import speed with DirectQuery freshness."

**Q: ADF vs Data Factory in Fabric?**
"Same data-integration heritage. **ADF is PaaS** (standalone Azure service); **Fabric Data Factory is SaaS** embedded in Fabric with Dataflows Gen2 and native OneLake integration."

**Q: Synapse dedicated vs serverless SQL pool?**
"**Dedicated** is a provisioned **MPP warehouse** for predictable high-performance workloads. **Serverless** queries lake files on-demand with T-SQL, pay-per-query — good for ad-hoc/exploration."

**Q: Synapse table distributions?**
"**Hash** for large fact tables (even spread on a key, less shuffle), **Round-Robin** for staging (even, no key), **Replicated** for small dimensions (copied to all nodes)."

---

## 12. Concept Q&A — MongoDB & Elasticsearch

**Q: How does MongoDB store data and what are indexes?**
"MongoDB stores **BSON documents** in **collections** with flexible schemas. **Indexes** (single-field, compound, text, etc.) speed up queries; without one, Mongo does a full collection scan. You pick indexes based on query patterns and sort needs."

**Q: Embedding vs referencing in MongoDB?**
"**Embed** related data that's read together to avoid joins (denormalized, fast reads). **Reference** when data is large, shared, or independently updated. It's a trade-off between read performance and duplication/consistency."

**Q: What makes Elasticsearch fast for search?**
"Its **inverted index** maps terms to documents, so full-text lookups are near-instant. **Analyzers** tokenize and normalize text at index time, and it's **near-real-time**. **Mappings** define field types (text vs keyword) and analyzers."

**Q: Query vs filter context in Elasticsearch?**
"**Query context** scores relevance ('how well does this match'). **Filter context** is a yes/no match (e.g., a date range or status) — it's cacheable and faster because it skips scoring."

**Q: How do MongoDB and Elasticsearch work together?**
"**MongoDB as the system of record**, **Elasticsearch as the search layer** — structured documents live in Mongo, and searchable fields are indexed into Elasticsearch, kept in sync via a pipeline. Common in catalog/content-search systems."

---

## 13. Concept Q&A — Airflow, Docker, CI/CD, GCP/BigQuery

**Q: What is Airflow and how does it compare to ADF?**
"**Airflow** orchestrates workflows as **DAGs** of tasks in Python, with scheduling, retries, sensors, and backfills. **ADF** is a managed, low-code Azure orchestrator. Airflow gives **code-first flexibility**; ADF integrates natively with Azure and is lower-maintenance. I'd use ADF in an all-Azure shop, Airflow for code-centric or multi-cloud orchestration."

**Q: Why containerize with Docker?**
"Docker packages an app with its dependencies into a portable **image**, so it runs **consistently** across dev/test/prod — no 'works on my machine' issues. Useful for reproducible pipeline/job environments."

**Q: How does CI/CD work for data pipelines?**
"Code lives in **Git**; a CI pipeline runs **build/tests/validation** on PRs; CD deploys to higher environments. For **ADF** that's Git integration + **ARM templates** with per-environment parameters; for **Databricks**, **Repos + Asset Bundles**."

**Q: BigQuery basics (you're certified)?**
"BigQuery is a **serverless, columnar, MPP** data warehouse. You optimize cost/performance with **partitioning and clustering**, avoiding `SELECT *`, and pruning partitions. It separates storage and compute and bills by data scanned (on-demand) or slots."

**Q: BigQuery partitioning vs clustering?**
"**Partitioning** splits a table by a column (often date) so queries scan only relevant partitions. **Clustering** sorts data within partitions by chosen columns for further pruning. Together they cut bytes scanned and cost."

---

## 14. Data Modeling & Warehousing

**Q: Star vs snowflake schema?**
"**Star** = central **fact** table with **denormalized dimensions** — fewer joins, fast for BI. **Snowflake** normalizes dimensions into sub-tables — less redundancy, more joins. I default to star for analytics."

**Q: Fact vs dimension table?**
"**Fact** holds measurable events/metrics at a defined **grain**, with foreign keys to dimensions. **Dimension** holds descriptive attributes (customer, product, date) for filtering and grouping."

**Q: SCD Type 1 vs Type 2?**
"**Type 1** overwrites the old value (no history). **Type 2** keeps history by inserting a **new row** with `start_date`, `end_date`, and `is_current` flags. I implement Type 2 with Delta **`MERGE`** — expire the old row, insert the new version."

**Q: OLTP vs OLAP?**
"**OLTP** = many small transactional writes, normalized (Azure SQL). **OLAP** = read-heavy analytical queries over aggregated, denormalized data (Synapse / lakehouse Gold)."

---

## 15. Scenario / Design Questions

**Q: Design a pipeline to ingest daily vendor files into ADLS, transform via medallion, and serve to Power BI.**

**Answer:**
"I'd use an **event-based or schedule trigger** in ADF to detect the file, **Copy** it into **Bronze** (raw Delta). A **Databricks** notebook cleans and dedupes into **Silver** with schema enforcement, then aggregates into **Gold** using `MERGE`. ADF orchestrates the dependencies with **retries and failure alerts**. For incrementals I use a **watermark or Auto Loader**. Gold tables are served via a **SQL endpoint / Fabric Direct Lake** to **Power BI**. I'd add **data-quality checks**, monitoring via Azure Monitor, and run it on a **job cluster** for cost. Security through **Key Vault + managed identity**."

**Q: A nightly job failed mid-run and left partial data. How does your design prevent corruption?**

**Answer:**
"**Delta's atomic commits** mean a partial write isn't visible — a failed write doesn't commit. Combined with **idempotent `MERGE`** and **checkpoints/watermarks**, I can simply **re-trigger** the job and it resumes cleanly without duplicating or corrupting data. Retries and alerts handle it automatically."

**Q: A large fact × small dimension join is slow. Optimize it.**

**Answer:**
"**Broadcast** the small dimension so the large fact isn't shuffled, **partition/bucket** the fact on the join key, **cache** the dimension if reused, run **OPTIMIZE/Z-Order** on the Delta tables, and enable **AQE**. I'd confirm the change in the **physical plan** showing a BroadcastHashJoin."

**Q: The source has no reliable modified-date for incremental loads. What do you do?**

**Answer:**
"I'd use **Change Tracking/CDC** if the source supports it. Otherwise, a **full extract into staging plus a hash comparison** to detect changed rows, then **`MERGE`** only the differences. As a last resort, full reload into a partition that's overwritten idempotently."

**Q: How would you enforce data quality across Bronze→Gold?**

**Answer:**
"Schema enforcement at Silver, **null/duplicate/range checks**, referential validation, and **reconciliation counts** between layers. In Databricks I can use **Delta Live Tables expectations** to drop/quarantine or fail on bad records, and alert on threshold breaches. Bad records go to a **quarantine table** for review rather than silently dropping."

---

## 16. Behavioral Questions

**Q: Tell me about a challenging problem you solved.**
"The **Talend-to-Databricks migration** was the most challenging. *(Situation)* We had to modernize a large legacy ETL estate with no automated converter. *(Task)* I had to re-implement each job in PySpark while guaranteeing identical output. *(Action)* I took a phased approach, rebuilt the logic as distributed Spark transforms, and ran parallel reconciliation on counts/checksums before each cut-over. *(Result)* We migrated successfully with validated parity and a more scalable, cost-efficient platform."

**Q: A time you improved performance.**
"*(Situation)* Spark jobs in our medallion pipeline were running long and costing DBUs. *(Task)* Improve throughput without changing outputs. *(Action)* I diagnosed shuffles/skew/small files in the Spark UI and applied broadcast joins, OPTIMIZE/Z-Order, repartitioning, AQE, Photon, and caching. *(Result)* Average batch runtime dropped ~30%, lowering cost and improving SLA reliability."

**Q: A time you disagreed with a teammate on design.**
"On a pipeline I favored an **idempotent MERGE-based incremental** while a teammate wanted full reloads for simplicity. *(Action)* I laid out the cost and runtime impact at scale and built a quick proof showing the incremental approach was far cheaper and still reliable. *(Result)* We adopted the incremental design, and I documented it so the team could reuse the pattern. I made sure the disagreement stayed about trade-offs, not personal."

**Q: How do you prioritize when multiple pipelines break at once?**
"I triage by **business impact and SLA** — what feeds critical downstream reporting first — communicate status to stakeholders, fix or mitigate the highest-impact one (often by re-triggering an idempotent run), then root-cause the rest. Clear communication matters as much as the fix."

**Q: A mistake you made and learned from.**
"Early on I deployed a transformation without enough **edge-case testing** and a null in an unexpected field caused bad aggregates downstream. I owned it, fixed it quickly, and added it to my standard **validation checks**. Since then I bake schema/null/dup checks into every pipeline by default."

**Q: Why are you looking to move / where in 3 years?**
"I want to work on **larger-scale, more modern data platforms** and deepen my expertise — ideally toward a **senior/lead data engineer** role owning architecture. I keep current via certifications and hands-on projects, and I'm looking for a team where I can grow that ownership."

---

## 17. Honest "Depth" Answers for Lighter Skills

Use these when probed on something you've used lightly — honesty + conceptual grasp beats bluffing.

**Q: How deep is your GCP/BigQuery experience?**
"My **production depth is on Azure** — Databricks, ADF, PySpark, Delta. On GCP I hold the **Professional Data Engineer and Database Engineer** certifications and have done **hands-on labs and POCs** with BigQuery, Dataflow, and Cloud Storage. I understand the architecture and equivalents well and can ramp quickly, but I'd describe my day-to-day production work as Azure-centric."

**Q: How much have you used Microsoft Fabric?**
"I've worked with it at a **working/exploratory level** — I understand **OneLake, Lakehouse vs Warehouse, Direct Lake, and how Fabric Data Factory differs from ADF as SaaS**. My heavy production hours are in ADF/Databricks, but I'm comfortable with where Fabric fits and the migration story."

**Q: How have you used Generative AI?**
"Mainly as a **productivity tool** — generating and reviewing code/SQL, documentation, and speeding up boilerplate. I'm also familiar with the **RAG pattern** (embeddings + vector search), which connects to my Elasticsearch experience. I'd be glad to go deeper if the role involves GenAI data pipelines."

**Principle:** Depth on your **core stack** wins; for peripheral tools, show **conceptual understanding + eagerness to learn** and never overclaim.

---

## 18. Questions to Ask the Interviewer

- What does your current data platform look like, and what's the biggest pain point you're hoping this role solves?
- Are you on **Unity Catalog / Delta Live Tables / Microsoft Fabric**, or migrating toward them?
- How is the team structured across data engineering, analytics, and platform?
- What does **success in the first six months** look like?
- How mature is your **CI/CD, testing, and data-quality** tooling for pipelines?
- How do you balance **performance, governance, and cost** at scale?

---

## 19. Final Checklist

The day before:

- [ ] Rehearse the **90-second resume walkthrough** (Innoplexus → SLO → Accenture, growing scope).
- [ ] Lock in **STAR stories**: Talend→Databricks migration, Spark 30% optimization, SLO automation, data-quality save, a design disagreement.
- [ ] Be able to **whiteboard your medallion architecture** and the **ADF watermark pattern**.
- [ ] Memorize your **Spark optimization toolkit**: broadcast, OPTIMIZE/Z-Order, partitioning, AQE, Photon, cache, salting.
- [ ] Drill **live SQL**: 2nd-highest salary, dedupe-latest, top-N per group, running total.
- [ ] Drill **live PySpark**: read→transform→write Delta, `MERGE` upsert, window functions.
- [ ] Prepare **honest depth statements** for GCP, Fabric, GenAI, Mongo/ES.
- [ ] Prepare **3 questions** to ask them.
- [ ] Review your concept guides (Python, SQL, PySpark, Databricks, Azure DE).

**Mindset:** You have a strong, coherent Azure DE profile with a clear migration + optimization narrative and solid certifications. Lead with **real project stories**, **quantify honestly**, and go **deep on your core stack**. That's exactly what interviewers want from a four-year data engineer.

---

