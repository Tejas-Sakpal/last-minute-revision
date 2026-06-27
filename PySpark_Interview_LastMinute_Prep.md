# PySpark Interview — Last-Minute Prep Guide

> **How to use this:** Skim the **Theory** for crisp definitions, read the **Code** to see it in action, and rehearse the **Interview Q&A** out loud. Bold lines are the one-liners interviewers want to hear. Built for a 2–4 hour final revision.
>
> *Examples target Spark 3.x with the DataFrame API. Assume `spark` is an active `SparkSession`.*

---

## Table of Contents

1. [Spark Fundamentals & Architecture](#1-spark-fundamentals--architecture)
2. [RDD vs DataFrame vs Dataset](#2-rdd-vs-dataframe-vs-dataset)
3. [SparkSession & SparkContext](#3-sparksession--sparkcontext)
4. [Transformations vs Actions (Lazy Evaluation)](#4-transformations-vs-actions)
5. [Narrow vs Wide Transformations & Shuffle](#5-narrow-vs-wide-transformations--shuffle)
6. [Creating & Inspecting DataFrames](#6-creating--inspecting-dataframes)
7. [Core DataFrame Operations](#7-core-dataframe-operations)
8. [Joins](#8-joins)
9. [Aggregations & GroupBy](#9-aggregations--groupby)
10. [Window Functions](#10-window-functions)
11. [Handling Nulls, Duplicates & Schema](#11-handling-nulls-duplicates--schema)
12. [UDFs vs Built-in Functions](#12-udfs-vs-built-in-functions)
13. [Caching, Persistence & Checkpointing](#13-caching-persistence--checkpointing)
14. [Partitioning & Performance Optimization](#14-partitioning--performance-optimization)
15. [Joins Optimization: Broadcast & Skew](#15-joins-optimization-broadcast--skew)
16. [Spark SQL & Reading/Writing Data](#16-spark-sql--readingwriting-data)
17. [Common Coding Programs (with solutions)](#17-common-coding-programs)
18. [Tricky / Gotcha Questions](#18-tricky--gotcha-questions)
19. [Rapid-Fire One-Liners](#19-rapid-fire-one-liners)
20. [Last-Minute Checklist](#20-last-minute-checklist)

---

## 1. Spark Fundamentals & Architecture

### Theory

**Apache Spark** is a distributed, in-memory data processing engine for large-scale data. **PySpark** is its Python API.

**Why Spark over MapReduce:** in-memory computation (vs disk-heavy MapReduce), lazy DAG optimization, unified engine (batch + streaming + SQL + ML + graph).

**Architecture (must know):**

- **Driver** — runs your `main()`, builds the **DAG**, creates the **SparkSession**, and schedules work. Hosts the **DAG Scheduler** and **Task Scheduler**.
- **Cluster Manager** — allocates resources (YARN, Kubernetes, Mesos, or Standalone).
- **Executors** — worker processes on nodes that run **tasks** and store data (cache) in memory/disk.
- **Job → Stages → Tasks:** an **action** triggers a **Job**, split into **Stages** at shuffle boundaries, each Stage = a set of **Tasks** (one per partition).

**Key abstractions:** **RDD** (low-level), **DataFrame** (structured, optimized via **Catalyst** + **Tungsten**), **Dataset** (typed, JVM only — not in Python).

### Code

```python
from pyspark.sql import SparkSession

spark = (SparkSession.builder
         .appName("InterviewPrep")
         .master("local[*]")          # use all cores locally
         .config("spark.sql.shuffle.partitions", "200")
         .getOrCreate())

print(spark.version)
```

### Interview Q&A

**Q: Explain Spark's architecture.**
A **driver** coordinates execution: it builds a DAG of transformations, requests resources from the **cluster manager**, and schedules **tasks** onto **executors** running on worker nodes. Executors do the actual computation and hold cached data.

**Q: What is a DAG in Spark?**
A **Directed Acyclic Graph** of operations Spark builds from your transformations. The DAG scheduler splits it into **stages** at shuffle boundaries and optimizes execution; it's the basis of lazy evaluation and fault recovery (lineage).

**Q: Job vs Stage vs Task?**
An **action** triggers a **job**. A job is divided into **stages** separated by shuffles. Each stage runs as parallel **tasks**, one per partition — the smallest unit of execution.

**Q: What are Catalyst and Tungsten?**
**Catalyst** is Spark SQL's query optimizer (logical → optimized → physical plan). **Tungsten** is the execution engine that optimizes memory and CPU (off-heap memory, code generation). Together they make DataFrames faster than raw RDDs.

---

## 2. RDD vs DataFrame vs Dataset

### Theory

| Feature | RDD | DataFrame | Dataset |
|---|---|---|---|
| Abstraction | Low-level distributed objects | Distributed rows with **schema** | Typed rows |
| Schema | No | Yes | Yes |
| Optimization | None | **Catalyst + Tungsten** | Catalyst + Tungsten |
| API | Functional (map/filter) | SQL-like (select/filter) | Typed (compile-time safe) |
| In PySpark? | Yes | **Yes (preferred)** | **No** (Scala/Java only) |
| Type safety | Compile-time (limited) | Runtime | Compile-time |

**RDD** = **R**esilient (fault-tolerant via lineage) **D**istributed **D**ataset — immutable, partitioned collection. Use only when you need fine-grained control or unstructured data.

**Rule of thumb:** in PySpark, **prefer DataFrames** — they're optimized, concise, and interoperable with Spark SQL.

### Code

```python
# RDD
rdd = spark.sparkContext.parallelize([1, 2, 3, 4])
print(rdd.map(lambda x: x * 2).collect())     # [2, 4, 6, 8]

# DataFrame (preferred)
df = spark.createDataFrame([(1, "a"), (2, "b")], ["id", "label"])
df.show()

# Convert between them
rdd2 = df.rdd                                  # DataFrame -> RDD
df2  = rdd.map(lambda x: (x,)).toDF(["num"])   # RDD -> DataFrame
```

### Interview Q&A

**Q: RDD vs DataFrame — which and why?**
A **DataFrame** has a schema and benefits from the **Catalyst optimizer** and **Tungsten**, so it's faster and more concise. An **RDD** is lower-level with no built-in optimization. Use DataFrames by default; drop to RDDs only for fine-grained or unstructured control.

**Q: What does "resilient" mean in RDD?**
Fault tolerance via **lineage** — Spark records the sequence of transformations (the DAG), so if a partition is lost, it can be **recomputed** from the source rather than replicated.

**Q: Why is there no Dataset API in PySpark?**
Datasets rely on **compile-time type safety** via JVM types, which Python (dynamically typed) can't provide. Python uses DataFrames (= `Dataset[Row]` conceptually).

**Q: Are RDDs/DataFrames mutable?**
No — they're **immutable**. Every transformation produces a **new** RDD/DataFrame; this enables lineage-based fault tolerance.

---

## 3. SparkSession & SparkContext

### Theory

- **SparkContext** — the original entry point (Spark 1.x); connection to the cluster, used to create RDDs.
- **SparkSession** — the **unified entry point** (Spark 2.0+) for DataFrame, SQL, and streaming. It **wraps** SparkContext, SQLContext, and HiveContext. Access the underlying context via `spark.sparkContext`.

### Code

```python
spark = SparkSession.builder.appName("app").getOrCreate()
sc = spark.sparkContext           # underlying SparkContext
sc.setLogLevel("ERROR")
```

### Interview Q&A

**Q: SparkSession vs SparkContext?**
**SparkContext** is the low-level connection to the cluster (creates RDDs). **SparkSession** (2.0+) is the single unified entry point for DataFrames, SQL, and streaming, and internally holds a SparkContext. Today you create a SparkSession; it gives you everything.

**Q: How many SparkContexts can run per JVM?**
By default, **only one** active SparkContext per JVM.

---

## 4. Transformations vs Actions

### Theory

**Lazy evaluation** is central to Spark. Operations split into:

- **Transformations** — define a new DataFrame/RDD but **don't execute** (lazy). They just extend the DAG. E.g. `select`, `filter`, `withColumn`, `groupBy`, `join`, `map`, `flatMap`, `distinct`, `orderBy`.
- **Actions** — **trigger** execution of the DAG and return a result or write data. E.g. `show`, `collect`, `count`, `take`, `first`, `write`, `foreach`, `reduce`.

**Why lazy?** Spark can optimize the whole chain (predicate pushdown, combining steps, minimizing shuffles) before running anything.

### Code

```python
df = spark.range(1, 1000000)

# transformations - nothing runs yet
filtered = df.filter(df.id % 2 == 0).withColumn("doubled", df.id * 2)

# action - NOW the DAG executes
print(filtered.count())           # triggers computation
filtered.show(5)                  # another action -> recomputes unless cached
```

### Interview Q&A

**Q: Transformation vs action?**
A **transformation** lazily builds a new dataset and is added to the DAG without running. An **action** triggers actual computation and returns a value to the driver or writes output. Nothing runs until an action is called.

**Q: What is lazy evaluation and why is it useful?**
Spark defers execution until an action, building a DAG it can **optimize as a whole** — reordering filters, pushing predicates to the source, and minimizing shuffles — improving performance and avoiding wasted work.

**Q: Give examples of each.**
Transformations: `filter`, `select`, `map`, `groupBy`, `join`. Actions: `count`, `collect`, `show`, `take`, `write`, `first`.

**Q: Is `collect()` safe?**
Risky — it pulls **all** data to the driver and can cause OOM on large datasets. Prefer `show()`, `take(n)`, or writing to storage.

---

## 5. Narrow vs Wide Transformations & Shuffle

### Theory

- **Narrow transformation** — each input partition contributes to **one** output partition; **no data movement** across the network. E.g. `map`, `filter`, `union`, `withColumn`. Fast, pipelined within a stage.
- **Wide transformation** — input partitions feed **multiple** output partitions, requiring a **shuffle** (data redistributed across executors/network). E.g. `groupBy`, `join`, `distinct`, `repartition`, `reduceByKey`, `orderBy`. Expensive — creates a new **stage**.

**Shuffle** = the costly process of redistributing data across partitions (disk I/O + network + serialization). Minimizing shuffles is the #1 performance lever.

### Code

```python
# Narrow - no shuffle
df.filter(df.age > 18)
df.withColumn("bonus", df.salary * 0.1)

# Wide - triggers shuffle (and a new stage)
df.groupBy("dept").count()
df.join(other, "id")
df.orderBy("salary")
```

### Interview Q&A

**Q: Narrow vs wide transformation?**
A **narrow** transformation maps each parent partition to a single child partition with no network movement (e.g. `map`, `filter`). A **wide** transformation requires data from multiple partitions, triggering a **shuffle** and a new stage (e.g. `groupBy`, `join`).

**Q: What is a shuffle and why is it expensive?**
A **shuffle** redistributes data across partitions/executors so related keys land together. It's costly due to **disk writes, network transfer, and serialization**, and it creates stage boundaries. Reducing shuffles is key to performance.

**Q: How do stages relate to shuffles?**
Spark creates a new **stage** at each shuffle (wide transformation) boundary. Narrow transformations are pipelined together within a single stage.

---

## 6. Creating & Inspecting DataFrames

### Theory

Create DataFrames from collections, files (CSV/JSON/Parquet), RDDs, or databases. Inspect with `show`, `printSchema`, `dtypes`, `columns`, `describe`.

### Code

```python
from pyspark.sql.types import StructType, StructField, StringType, IntegerType

data = [(1, "Asha", 90000), (2, "Ben", 60000), (3, "Cy", 60000)]
schema = StructType([
    StructField("id", IntegerType(), True),
    StructField("name", StringType(), True),
    StructField("salary", IntegerType(), True),
])
df = spark.createDataFrame(data, schema)

df.show()
df.printSchema()                  # column names + types
print(df.columns)                 # ['id', 'name', 'salary']
print(df.count())                 # 3
df.describe().show()              # summary stats
```

### Interview Q&A

**Q: How do you define a schema explicitly and why?**
With `StructType`/`StructField`. Explicit schemas **avoid the cost of schema inference**, prevent wrong type guesses, and make pipelines robust to bad data.

**Q: `printSchema()` vs `describe()`?**
`printSchema()` shows column **names and data types**; `describe()`/`summary()` shows **statistics** (count, mean, stddev, min, max) for columns.

---

## 7. Core DataFrame Operations

### Theory

Common ops: `select`, `filter`/`where`, `withColumn` (add/replace col), `withColumnRenamed`, `drop`, `distinct`, `orderBy`/`sort`, `limit`, `alias`. Column expressions use `col()`, `lit()`, and `F` functions.

### Code

```python
from pyspark.sql import functions as F
from pyspark.sql.functions import col

df.select("name", "salary").show()
df.filter(col("salary") > 60000).show()
df.where((col("salary") > 50000) & (col("name") != "Ben")).show()

df.withColumn("tax", col("salary") * 0.2).show()      # add column
df.withColumnRenamed("salary", "pay").show()          # rename
df.drop("id").show()                                  # remove column
df.orderBy(col("salary").desc()).show()               # sort
df.select(F.upper(col("name")).alias("NAME")).show()  # function + alias

# Conditional column (CASE WHEN)
df.withColumn("band", F.when(col("salary") >= 80000, "High")
                       .when(col("salary") >= 50000, "Mid")
                       .otherwise("Low")).show()
```

### Interview Q&A

**Q: `select` vs `withColumn`?**
`select` projects a chosen set of columns/expressions. `withColumn` **adds or replaces a single column** while keeping all others. Avoid chaining many `withColumn` calls in a loop — use `select` with all expressions for better performance.

**Q: `filter` vs `where`?**
They're **identical** — `where` is an alias of `filter`. Both accept a column expression or SQL string.

**Q: How do you implement CASE WHEN logic?**
Use `F.when(condition, value).when(...).otherwise(default)` inside `withColumn` or `select`.

**Q: Difference between `sort`/`orderBy` and why is it a wide op?**
They're equivalent for global sorting. It's a **wide** transformation because globally ordering data requires a **shuffle** to range-partition rows.

---

## 8. Joins

### Theory

`df1.join(df2, on="key", how="<type>")`. Join types: `inner` (default), `left`/`left_outer`, `right`/`right_outer`, `full`/`outer`, `left_semi` (rows in left **with** a match, left cols only), `left_anti` (rows in left with **no** match), `cross`.

### Code

```python
emp  = spark.createDataFrame([(1,"Asha",10),(2,"Ben",10),(3,"Cy",99)], ["id","name","dept_id"])
dept = spark.createDataFrame([(10,"Eng"),(20,"Sales")], ["dept_id","dept_name"])

emp.join(dept, "dept_id", "inner").show()      # only matches
emp.join(dept, "dept_id", "left").show()       # all employees, null dept if none

# Anti-join: employees with no matching department
emp.join(dept, "dept_id", "left_anti").show()  # Cy (dept 99)

# Semi-join: employees that DO have a department (left cols only)
emp.join(dept, "dept_id", "left_semi").show()  # Asha, Ben

# Join on different column names
emp.join(dept, emp.dept_id == dept.dept_id, "inner")
```

### Interview Q&A

**Q: What join types does PySpark support?**
inner, left/right/full outer, **left_semi** (keeps left rows that have a match, returns left columns only), **left_anti** (keeps left rows with no match), and cross.

**Q: Semi-join vs inner join?**
A **left_semi** join returns only **left-table columns** and one row per left match (like a `WHERE EXISTS`). An inner join returns columns from **both** tables and can multiply rows on one-to-many matches.

**Q: How do you avoid duplicate join-key columns?**
Join using a **string/list key** (`on="dept_id"`) instead of a boolean condition — Spark keeps a single key column. With a condition you get both columns and must drop one.

---

## 9. Aggregations & GroupBy

### Theory

`groupBy(...).agg(...)` with `F.count`, `F.sum`, `F.avg`, `F.min`, `F.max`, `F.countDistinct`, `F.collect_list`, `F.collect_set`. `groupBy` is a **wide** (shuffle) operation. `agg` without `groupBy` aggregates the whole DataFrame.

### Code

```python
from pyspark.sql import functions as F

df.groupBy("dept").agg(
    F.count("*").alias("headcount"),
    F.avg("salary").alias("avg_sal"),
    F.max("salary").alias("top_sal"),
    F.collect_list("name").alias("members")
).show()

# Filter groups (HAVING equivalent)
(df.groupBy("dept")
   .agg(F.avg("salary").alias("avg_sal"))
   .filter(F.col("avg_sal") > 60000)
   .show())

# Pivot
df.groupBy("dept").pivot("gender").agg(F.avg("salary")).show()
```

### Interview Q&A

**Q: How do you do HAVING in PySpark?**
Apply `.filter()`/`.where()` **after** the `groupBy().agg()` — the post-aggregation filter is the HAVING equivalent.

**Q: `count("*")` vs `count("col")`?**
`count("*")` counts all rows; `count("col")` counts **non-null** values in that column. Use `countDistinct("col")` for unique values.

**Q: What is `collect_list` vs `collect_set`?**
Both gather a column's values per group into an array. `collect_list` keeps **duplicates** and order; `collect_set` returns **distinct** values (unordered).

---

## 10. Window Functions

### Theory

Window functions compute across a set of rows **related to the current row** without collapsing them — used for ranking, running totals, and row comparisons. Define a `Window` spec with `partitionBy` and `orderBy`.

**Functions:** `row_number()`, `rank()`, `dense_rank()`, `ntile(n)`, `lag()`, `lead()`, and aggregates (`sum`, `avg`) over a window.

**rank vs dense_rank vs row_number** (classic): on ties at top — `row_number` → 1,2,3 (unique); `rank` → 1,1,3 (skips); `dense_rank` → 1,1,2 (no gaps).

### Code

```python
from pyspark.sql.window import Window
from pyspark.sql import functions as F

w = Window.partitionBy("dept").orderBy(F.col("salary").desc())

df.withColumn("rnk", F.dense_rank().over(w)).show()        # rank within dept

# Top earner per department
df.withColumn("rn", F.row_number().over(w)) \
  .filter(F.col("rn") == 1).show()

# Running total
w2 = Window.partitionBy("dept").orderBy("id") \
           .rowsBetween(Window.unboundedPreceding, Window.currentRow)
df.withColumn("running_total", F.sum("salary").over(w2)).show()

# Previous row value
df.withColumn("prev_sal", F.lag("salary", 1).over(w)).show()
```

### Interview Q&A

**Q: Window function vs groupBy?**
`groupBy` **collapses** rows into one per group. A window function **keeps every row** and adds the computed value alongside — letting you show detail and aggregate together (e.g. each employee plus their dept rank).

**Q: row_number vs rank vs dense_rank?**
All rank within an ordered window. `row_number` gives a **unique** sequential number; `rank` gives ties the same rank and **skips**; `dense_rank` gives ties the same rank with **no gaps**.

**Q: How do you get the top-N per group?**
Add `row_number()`/`rank()` over a `Window.partitionBy(group).orderBy(metric desc)` and `filter` where rank ≤ N.

**Q: What's the difference between `rowsBetween` and `rangeBetween`?**
`rowsBetween` is a physical offset (count of rows); `rangeBetween` is based on the **value** of the ordering column. Use `rowsBetween` for running totals.

---

## 11. Handling Nulls, Duplicates & Schema

### Theory

- **Nulls:** `df.na.drop()`, `df.na.fill(value)`, `F.coalesce(c1, c2)`, `isNull()`/`isNotNull()`.
- **Duplicates:** `df.distinct()`, `df.dropDuplicates(["col"])`.
- **Schema/cast:** `col.cast("int")`, `df.schema`, `printSchema()`.

### Code

```python
from pyspark.sql import functions as F

df.na.drop().show()                          # drop rows with any null
df.na.drop(subset=["salary"]).show()         # only if salary null
df.na.fill({"salary": 0, "name": "Unknown"}).show()
df.withColumn("phone", F.coalesce("phone", F.lit("N/A"))).show()

df.dropDuplicates(["name"]).show()           # de-dup by column
df.withColumn("salary", F.col("salary").cast("double")).printSchema()
```

### Interview Q&A

**Q: How do you handle null values?**
Drop them (`na.drop`), fill defaults (`na.fill`), or substitute with `coalesce`. Filter with `isNull()`/`isNotNull()`. Choice depends on whether missing data should be removed or imputed.

**Q: `distinct()` vs `dropDuplicates()`?**
`distinct()` dedups on the **entire row**. `dropDuplicates(cols)` dedups based on the **specified columns** (keeping the first occurrence).

**Q: How do you change a column's type?**
`df.withColumn("c", col("c").cast("type"))`. Invalid casts become **null** rather than erroring.

---

## 12. UDFs vs Built-in Functions

### Theory

- **Built-in functions** (`pyspark.sql.functions`) run inside the JVM and are **Catalyst-optimized** — always prefer them.
- **UDF (User Defined Function)** — custom Python function wrapped for column use. **Costly**: data is serialized between JVM and Python, and UDFs are a **black box** to Catalyst (no optimization, no pushdown).
- **Pandas UDFs (vectorized)** — use Apache Arrow to process batches, far faster than row-at-a-time UDFs.

**Rule:** avoid UDFs when a built-in exists; if you must, prefer **pandas UDFs**.

### Code

```python
from pyspark.sql.functions import udf, pandas_udf
from pyspark.sql.types import StringType

# Regular UDF (slow - avoid if possible)
@udf(returnType=StringType())
def grade(salary):
    return "High" if salary >= 80000 else "Low"

df.withColumn("grade", grade("salary")).show()

# Prefer built-in equivalent (fast)
df.withColumn("grade", F.when(F.col("salary") >= 80000, "High").otherwise("Low")).show()

# Vectorized pandas UDF (fast)
@pandas_udf("double")
def add_tax(s):
    return s * 1.2
df.withColumn("with_tax", add_tax("salary")).show()
```

### Interview Q&A

**Q: Why avoid Python UDFs?**
They cause **serialization overhead** between the JVM and Python and are **opaque to Catalyst**, so Spark can't optimize or push down predicates through them. Built-in functions run natively in the JVM and are much faster.

**Q: What's a pandas/vectorized UDF?**
A UDF that uses **Apache Arrow** to transfer and process data in **batches** (as pandas Series) rather than row-by-row, dramatically reducing overhead compared to standard UDFs.

---

## 13. Caching, Persistence & Checkpointing

### Theory

Because Spark is lazy, a DataFrame used by **multiple actions** is **recomputed** each time. Caching avoids that.

- **`cache()`** — persist with the default storage level (`MEMORY_AND_DISK` for DataFrames).
- **`persist(level)`** — choose the storage level (`MEMORY_ONLY`, `MEMORY_AND_DISK`, `DISK_ONLY`, with `_SER`/`_2` variants).
- **`unpersist()`** — free it.
- **Checkpointing** — saves an RDD/DataFrame to **reliable storage (HDFS)** and **truncates the lineage**. Used for very long lineages (avoids stack/recompute issues) and streaming fault tolerance. Requires `sc.setCheckpointDir(...)`.

**Cache vs checkpoint:** cache keeps **lineage** (can recompute if lost, stored in memory/local disk); checkpoint **cuts lineage** and writes to reliable storage (survives failures, but is an action-like write).

### Code

```python
df_filtered = df.filter(df.salary > 50000)
df_filtered.cache()               # mark for caching
df_filtered.count()               # action materializes the cache
df_filtered.show()                # reuses cache - no recompute

from pyspark import StorageLevel
df_filtered.persist(StorageLevel.MEMORY_AND_DISK)
df_filtered.unpersist()

spark.sparkContext.setCheckpointDir("/tmp/ckpt")
df_long = df_filtered.checkpoint()    # truncates lineage
```

### Interview Q&A

**Q: When and why do you cache?**
When a DataFrame is **reused across multiple actions**, caching stores it in memory/disk so Spark doesn't recompute the whole DAG each time — a major speedup for iterative or branching pipelines.

**Q: `cache()` vs `persist()`?**
`cache()` is shorthand for `persist()` with the default storage level. `persist()` lets you **choose** the level (memory only, memory+disk, disk only, serialized, replicated).

**Q: Cache vs checkpoint?**
**Cache** stores data (in memory/local disk) but **keeps lineage**, so it can recompute on loss. **Checkpoint** writes to **reliable storage** and **truncates lineage**, breaking long dependency chains and aiding fault tolerance — but it's more expensive.

**Q: Does `cache()` execute immediately?**
No — it's **lazy**. The data is materialized only when the **next action** runs.

---

## 14. Partitioning & Performance Optimization

### Theory

**Partitions** are the units of parallelism — each is processed by one task. Too few = under-utilized cluster; too many = scheduling overhead and small-file problems.

- **`repartition(n)`** — **full shuffle** to exactly `n` partitions (can increase or decrease); good for balancing skew.
- **`coalesce(n)`** — **narrow**, merges partitions **without a full shuffle**; only **decreases** count (efficient before writing output).
- **`partitionBy(col)`** on write — physically partitions output files by column (enables partition pruning on read).
- **`spark.sql.shuffle.partitions`** — number of partitions after a shuffle (default 200; tune for data size).

**Top optimization techniques (be ready to list):**
1. Prefer **DataFrames** over RDDs (Catalyst).
2. **Minimize shuffles** (avoid unnecessary `groupBy`/`join`/`distinct`).
3. **Broadcast** small tables in joins.
4. **Cache/persist** reused data.
5. **Filter early / select only needed columns** (predicate & projection pushdown).
6. Use efficient formats — **Parquet** (columnar, compressed) over CSV/JSON.
7. **Tune partitions** (`repartition`/`coalesce`, `shuffle.partitions`).
8. Avoid **UDFs**; use built-ins.
9. Handle **data skew** (salting).
10. Enable **AQE (Adaptive Query Execution)** in Spark 3.

### Code

```python
print(df.rdd.getNumPartitions())            # inspect partition count

df_repart = df.repartition(8, "dept")       # full shuffle, balance by dept
df_coal   = df.coalesce(1)                   # merge to 1, no full shuffle

# Write partitioned Parquet (enables partition pruning)
df.write.mode("overwrite").partitionBy("dept").parquet("/tmp/out")

# Enable Adaptive Query Execution
spark.conf.set("spark.sql.adaptive.enabled", "true")
```

### Interview Q&A

**Q: `repartition` vs `coalesce`?**
`repartition(n)` does a **full shuffle** and can increase or decrease partitions, producing balanced partitions. `coalesce(n)` **only decreases** and avoids a full shuffle by merging existing partitions — cheaper, but can leave skewed partitions. Use `coalesce` to reduce partitions before writing.

**Q: How do you optimize a slow PySpark job?**
Minimize shuffles, broadcast small tables, cache reused data, filter/select early (pushdown), use Parquet, tune partition counts, avoid UDFs, address skew with salting, and enable **AQE**. Inspect the **Spark UI / `explain()`** to find the bottleneck.

**Q: What is data skew and how do you fix it?**
Skew is when a few keys hold a disproportionate share of data, overloading some tasks. Fix with **salting** (add a random prefix to spread the hot key), broadcasting the small side, or enabling **AQE skew join handling**.

**Q: What is AQE?**
**Adaptive Query Execution** (Spark 3) re-optimizes the plan **at runtime** using actual statistics — coalescing shuffle partitions, switching join strategies, and handling skew automatically.

**Q: Why Parquet over CSV?**
Parquet is **columnar, compressed, and carries a schema**, enabling **column pruning** and **predicate pushdown** — far less I/O than row-based CSV/JSON.

---

## 15. Joins Optimization: Broadcast & Skew

### Theory

When joining a **large** table with a **small** one, Spark can **broadcast** the small table to every executor, avoiding a shuffle of the large table — a **broadcast hash join**. Triggered automatically below `spark.sql.autoBroadcastJoinThreshold` (default 10 MB) or manually with `F.broadcast()`.

Join strategies Spark chooses among: **Broadcast Hash Join** (small side), **Sort-Merge Join** (large/large, default), **Shuffle Hash Join**.

### Code

```python
from pyspark.sql.functions import broadcast

# Force broadcast of the small dimension table
result = large_df.join(broadcast(small_df), "key")

# Inspect the chosen plan
result.explain()                  # look for BroadcastHashJoin
```

### Interview Q&A

**Q: What is a broadcast join and when to use it?**
When one table is **small enough to fit in memory**, Spark sends (broadcasts) it to all executors so the **large** table isn't shuffled. Use it for large-with-small joins (e.g. fact × dimension) — it eliminates the expensive shuffle.

**Q: What's the default join strategy?**
**Sort-Merge Join** for two large tables (both sides shuffled and sorted on the key). Spark prefers a **Broadcast Hash Join** when one side is under the broadcast threshold.

**Q: How do you check which join Spark used?**
Call `.explain()` (or view the Spark UI) and look for `BroadcastHashJoin`, `SortMergeJoin`, etc., in the physical plan.

---

## 16. Spark SQL & Reading/Writing Data

### Theory

Register a DataFrame as a temp view and run SQL with `spark.sql(...)`. Read/write with `spark.read` / `df.write` across formats (CSV, JSON, Parquet, ORC, JDBC). Use **save modes**: `overwrite`, `append`, `ignore`, `error`.

### Code

```python
df.createOrReplaceTempView("employees")
spark.sql("""
    SELECT dept_id, AVG(salary) AS avg_sal
    FROM employees
    GROUP BY dept_id
    HAVING AVG(salary) > 60000
""").show()

# Read
csv_df = spark.read.option("header", True).option("inferSchema", True).csv("data.csv")
pq_df  = spark.read.parquet("data.parquet")

# Write
df.write.mode("overwrite").parquet("/tmp/out.parquet")
df.write.mode("append").partitionBy("dept_id").parquet("/tmp/parts")
```

### Interview Q&A

**Q: How do you run SQL on a DataFrame?**
Register it as a view with `createOrReplaceTempView("name")`, then query via `spark.sql("SELECT ...")`. DataFrame API and SQL compile to the **same Catalyst plan** — performance is identical.

**Q: `createOrReplaceTempView` vs `createGlobalTempView`?**
A **temp view** is scoped to the current SparkSession. A **global temp view** lives in the `global_temp` database and is shared across sessions in the same application.

**Q: What are save modes?**
`overwrite` (replace), `append` (add), `ignore` (skip if exists), `error`/`errorifexists` (default — fail if path exists).

---

## 17. Common Coding Programs

The hands-on tasks interviewers most often ask. *(Assume `df` with `id, name, dept, salary`.)*

### 1. Word count (the classic)

```python
from pyspark.sql import functions as F
lines = spark.createDataFrame([("hello world",), ("hello spark",)], ["line"])
(lines.select(F.explode(F.split("line", " ")).alias("word"))
      .groupBy("word").count()
      .orderBy(F.desc("count")).show())
```

### 2. Second highest salary

```python
from pyspark.sql.window import Window
w = Window.orderBy(F.col("salary").desc())
(df.withColumn("rnk", F.dense_rank().over(w))
   .filter(F.col("rnk") == 2)
   .select("name", "salary").show())
```

### 3. Top earner per department

```python
w = Window.partitionBy("dept").orderBy(F.col("salary").desc())
(df.withColumn("rn", F.row_number().over(w))
   .filter(F.col("rn") == 1).show())
```

### 4. Remove duplicates

```python
df.dropDuplicates(["name"]).show()
```

### 5. Find duplicates

```python
(df.groupBy("name").count().filter(F.col("count") > 1).show())
```

### 6. Department-wise aggregation

```python
df.groupBy("dept").agg(
    F.count("*").alias("cnt"),
    F.avg("salary").alias("avg_sal")
).show()
```

### 7. Running total per department

```python
w = Window.partitionBy("dept").orderBy("id")
df.withColumn("running", F.sum("salary").over(w)).show()
```

### 8. Filter + new column (band)

```python
df.withColumn("band", F.when(F.col("salary") >= 80000, "High")
                        .otherwise("Low")).show()
```

### 9. Employees earning above department average

```python
w = Window.partitionBy("dept")
(df.withColumn("dept_avg", F.avg("salary").over(w))
   .filter(F.col("salary") > F.col("dept_avg"))
   .select("name", "salary", "dept_avg").show())
```

### 10. Pivot table

```python
df.groupBy("dept").pivot("band").agg(F.count("*")).show()
```

### 11. Explode array column

```python
arr = spark.createDataFrame([(1, ["a","b"]),(2, ["c"])], ["id","tags"])
arr.select("id", F.explode("tags").alias("tag")).show()
```

### 12. Convert RDD to DataFrame

```python
rdd = spark.sparkContext.parallelize([(1, "a"), (2, "b")])
rdd.toDF(["id", "label"]).show()
```

---

## 18. Tricky / Gotcha Questions

Know the *why*.

**1. Transformations don't run.**
`df.filter(...)` alone does nothing — no action, no execution. Forgetting this leads to "why is my code instant but `show()` is slow?"

**2. `collect()` on big data = OOM.**
It pulls everything to the driver. Use `take(n)`, `show()`, or write to storage.

**3. DataFrames are immutable.**
`df.withColumn(...)` returns a **new** DataFrame — it doesn't modify `df` in place. You must reassign.

**4. Caching is lazy.**
`df.cache()` doesn't store anything until an **action** materializes it.

**5. `repartition` always shuffles; `coalesce` doesn't (and can't increase partitions).**

**6. UDFs kill optimization.**
A Python UDF blocks Catalyst pushdown and adds serialization cost. Prefer built-ins.

**7. Nulls in joins/filters.**
`col != "x"` excludes nulls too (null comparisons are null/false). Use `isNull()` or `eqNullSafe()` (`<=>`) for null-safe equality.

**8. Default shuffle partitions = 200.**
On small data this creates 200 tiny tasks; tune `spark.sql.shuffle.partitions` (or use AQE).

**9. `withColumn` in a loop is slow.**
Each call adds a plan node; for many columns, build one `select` with all expressions.

**10. `count()` triggers full computation.**
It's an action — recomputes the DAG unless cached. Don't sprinkle `count()` for debugging on expensive pipelines.

**11. Order of operations matters for performance.**
Filter **before** join/aggregation to reduce shuffled data (Catalyst usually pushes this down, but be explicit with UDFs).

---

## 19. Rapid-Fire One-Liners

- **PySpark** = Python API for Apache Spark (distributed, in-memory).
- **RDD** = low-level, no optimization; **DataFrame** = schema + Catalyst (preferred); **Dataset** = JVM-only.
- **Driver** plans; **executors** run tasks; **cluster manager** allocates resources.
- **Job → Stages (split at shuffles) → Tasks (one per partition).**
- **Transformations** are lazy; **actions** trigger execution.
- **Narrow** = no shuffle (`map`,`filter`); **wide** = shuffle (`groupBy`,`join`).
- **Shuffle** = expensive data redistribution (disk + network).
- **Catalyst** = query optimizer; **Tungsten** = execution/memory engine.
- **`cache()`** = persist default level; **`persist()`** = choose level.
- **Cache** keeps lineage; **checkpoint** truncates lineage to reliable storage.
- **`repartition`** = full shuffle (up/down); **`coalesce`** = no full shuffle (down only).
- **Broadcast join** for large × small; avoids shuffling the big table.
- **Default join** = Sort-Merge Join.
- **Avoid UDFs**; prefer built-in `F.*`; if needed, use **pandas UDFs**.
- **`collect()`** pulls all to driver (OOM risk) — use `show`/`take`.
- **Parquet** > CSV (columnar, pushdown).
- **`distinct()`** = whole row; **`dropDuplicates(cols)`** = by columns.
- **`row_number`** unique, **`rank`** skips ties, **`dense_rank`** no gaps.
- **`eqNullSafe`/`<=>`** = null-safe equality.
- **AQE** = runtime re-optimization (Spark 3).
- **`explain()`** shows the physical plan (which join, scans, etc.).
- **SparkSession** = unified entry point (wraps SparkContext).

---

## 20. Last-Minute Checklist

Run through these the hour before your interview:

- [ ] Spark architecture: driver, executors, cluster manager, DAG, job/stage/task.
- [ ] RDD vs DataFrame vs Dataset (and why no Dataset in Python).
- [ ] Lazy evaluation; transformations vs actions (with examples).
- [ ] Narrow vs wide transformations; what a shuffle is and why it's costly.
- [ ] Core ops: select, filter, withColumn, when/otherwise, orderBy.
- [ ] All join types incl. semi/anti; avoiding duplicate key columns.
- [ ] groupBy/agg, HAVING equivalent, collect_list vs collect_set.
- [ ] Window functions: rank vs dense_rank vs row_number, top-N per group, running total.
- [ ] Null handling, distinct vs dropDuplicates, casting.
- [ ] Why avoid UDFs; pandas UDFs.
- [ ] cache vs persist vs checkpoint; lazy caching.
- [ ] repartition vs coalesce; partition tuning; shuffle.partitions.
- [ ] Broadcast joins; sort-merge default; data skew + salting; AQE.
- [ ] Spark SQL temp views; Parquet vs CSV; save modes.
- [ ] Be ready to write: word count, 2nd highest salary, top-per-group, running total, dedup, dept aggregation.

**Interview tips:** Always mention **lazy evaluation** and **minimizing shuffles** when asked about performance. Reach for **DataFrame API + built-in functions** over RDDs/UDFs. When asked to optimize, structure your answer as: reduce shuffle → broadcast small tables → cache reused data → tune partitions → right file format. Use `.explain()` to justify decisions.

---

