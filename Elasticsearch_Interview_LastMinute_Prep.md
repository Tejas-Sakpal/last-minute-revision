# Elasticsearch Interview — Last-Minute Prep Guide

> **How to use this:** Skim the **Theory** for crisp definitions, read the **Query/Program** to see it in action, and rehearse the **Interview Q&A** out loud. Bold lines are the one-liners interviewers want to hear. Built for a 2–4 hour final revision.
>
> *Examples use the Elasticsearch REST API (JSON over HTTP), as you'd run in Kibana Dev Tools. Concepts apply to Elasticsearch 7.x–8.x.*

---

## Table of Contents

1. [Elasticsearch Fundamentals](#1-elasticsearch-fundamentals)
2. [Core Concepts: Index, Document, Shard, Replica](#2-core-concepts)
3. [The Inverted Index](#3-the-inverted-index)
4. [Mapping & Data Types](#4-mapping--data-types)
5. [Analysis: Analyzers, Tokenizers, Filters](#5-analysis-analyzers-tokenizers-filters)
6. [Indexing Documents (CRUD)](#6-indexing-documents-crud)
7. [Query DSL: Full-Text vs Term-Level](#7-query-dsl-full-text-vs-term-level)
8. [Compound Queries & the bool Query](#8-compound-queries--the-bool-query)
9. [Relevance Scoring & Query vs Filter Context](#9-relevance-scoring--query-vs-filter-context)
10. [Aggregations](#10-aggregations)
11. [Cluster Architecture & Node Roles](#11-cluster-architecture--node-roles)
12. [Sharding, Replication & Routing](#12-sharding-replication--routing)
13. [Read & Write Path / Near Real-Time](#13-read--write-path--near-real-time)
14. [Text vs Keyword & Common Pitfalls](#14-text-vs-keyword--common-pitfalls)
15. [Performance & Optimization](#15-performance--optimization)
16. [The ELK / Elastic Stack](#16-the-elk--elastic-stack)
17. [Common Query Programs](#17-common-query-programs)
18. [Tricky / Gotcha Questions](#18-tricky--gotcha-questions)
19. [Rapid-Fire One-Liners](#19-rapid-fire-one-liners)
20. [Last-Minute Checklist](#20-last-minute-checklist)

---

## 1. Elasticsearch Fundamentals

### Theory

**Elasticsearch** is an open-source, **distributed, RESTful search and analytics engine** built on **Apache Lucene**. It stores data as **JSON documents** and is optimized for **full-text search**, **structured search**, and **analytics at scale** with near-real-time performance.

**Why it's fast for search:** it builds an **inverted index** (term → documents) instead of scanning rows, so full-text lookups are near-instant.

**Common use cases:** full-text/app search, **log and event analytics (ELK)**, observability/metrics, security analytics (SIEM), and as a search layer alongside a system of record (e.g., **MongoDB → Elasticsearch**).

**Key traits:** schema-flexible (dynamic mapping), horizontally scalable (sharding), highly available (replicas), distributed, document-oriented, RESTful (JSON API).

### Interview Q&A

**Q: What is Elasticsearch?**
A **distributed, RESTful search and analytics engine** built on **Lucene** that stores **JSON documents** and uses an **inverted index** for fast full-text and structured search at scale, with near-real-time results.

**Q: What is Elasticsearch built on, and what does it add?**
It's built on **Apache Lucene** (the underlying search library). Elasticsearch adds **distribution (sharding/replication), a REST/JSON API, clustering, near-real-time search, aggregations, and scalability** — turning Lucene into a production-grade distributed engine.

**Q: When would you use Elasticsearch over a relational database?**
For **full-text search** (relevance ranking, fuzzy/typo tolerance, autocomplete), **log/analytics** over huge volumes, and fast aggregations. Use a **relational DB** for transactional integrity, joins, and strong consistency — Elasticsearch is typically a **search/analytics layer**, not the system of record.

---

## 2. Core Concepts

### Theory

Relational analogy (loose — not exact):

| RDBMS | Elasticsearch |
|---|---|
| Database | **Index** |
| Table | (Index / type — types removed in 7.x) |
| Row | **Document** (JSON) |
| Column | **Field** |
| Schema | **Mapping** |

- **Document** — the basic unit; a JSON object with a unique **`_id`**, stored in an index.
- **Index** — a collection of documents with similar characteristics (like a database/table). Physically split into **shards**.
- **Shard** — a self-contained **Lucene index**; the unit of distribution and scaling. **Primary** shards hold the data; **replica** shards are copies.
- **Replica** — a copy of a primary shard for **high availability** and **read throughput**.
- **Node** — a single Elasticsearch server. **Cluster** — a group of nodes sharing data and workload.

> Note: **mapping types** (e.g., `_type`) were **deprecated in 6.x and removed in 7.x** — one index now holds one document type.

### Interview Q&A

**Q: Index vs document vs shard?**
A **document** is a single JSON record (with an `_id`). An **index** is a collection of related documents. A **shard** is a physical **Lucene index** — an index is split into shards to distribute and scale across nodes.

**Q: What's the difference between a primary and a replica shard?**
A **primary** shard holds the original data and handles writes. A **replica** is a **copy** of a primary on a different node — it provides **fault tolerance** (takes over if the primary's node fails) and **scales reads**. A replica is never on the same node as its primary.

**Q: What happened to "types" in Elasticsearch?**
Mapping **types were removed in 7.x**. Previously one index could hold multiple types; now the guidance is **one document type per index** (use separate indices instead).

---

## 3. The Inverted Index

### Theory

The **inverted index** is the data structure that makes Elasticsearch fast. Instead of mapping documents → words, it maps **each unique term → the list of documents containing it** (the postings list), plus term frequency/position info.

**Example:** for docs `{1: "the quick fox"}`, `{2: "the lazy dog"}`:

```
term    -> documents
the     -> [1, 2]
quick   -> [1]
fox     -> [1]
lazy    -> [2]
dog     -> [2]
```

A search for "fox" jumps straight to `[1]` — no scanning. Terms are produced by the **analyzer** at index time. For exact-match/structured data, fields are stored as **keywords** and may use **doc values** (a columnar structure) for sorting/aggregations.

### Interview Q&A

**Q: What is an inverted index and why is it fast?**
It's a structure mapping **each term to the documents that contain it** (a postings list). A full-text search looks up the term and instantly gets the matching documents — **no full scan** — which is why Elasticsearch search is near-instant even over huge datasets.

**Q: How are the terms in the inverted index created?**
By the **analyzer** at index time — it **tokenizes** and normalizes the text (lowercasing, removing stop words, stemming, etc.) into the terms that get indexed. The same analyzer is applied to the query so they match.

**Q: What are doc values?**
A **columnar, on-disk** structure storing field values per document, used for **sorting, aggregations, and scripting** (the inverted index is great for search but not for these). Enabled by default on non-text fields.

---

## 4. Mapping & Data Types

### Theory

**Mapping** is the **schema** of an index — a JSON definition of the fields, their **data types**, and how they're indexed/analyzed.

- **Dynamic mapping** — Elasticsearch **auto-detects** field types when you index new documents (convenient but can guess wrong, e.g., a date as text).
- **Explicit mapping** — you define types up front (recommended for production) to control analysis, avoid type drift, and save space.

**Common data types:** `text`, `keyword`, `integer`/`long`, `float`/`double`, `boolean`, `date`, `object` (nested JSON), `nested` (array of objects queried independently), `geo_point`, `ip`.

**Mappings can't be changed for existing fields** — to change a field type you must **create a new index and reindex**.

### Program

```json
PUT /products
{
  "mappings": {
    "properties": {
      "name":     { "type": "text" },
      "sku":      { "type": "keyword" },
      "price":    { "type": "double" },
      "in_stock": { "type": "boolean" },
      "created":  { "type": "date" },
      "tags":     { "type": "keyword" }
    }
  }
}
```

### Interview Q&A

**Q: What is a mapping?**
The **schema** of an index — it defines each field's **data type** and how it's **indexed and analyzed**. It controls how documents are stored, searched, and aggregated.

**Q: Dynamic vs explicit mapping?**
**Dynamic** mapping lets Elasticsearch **auto-infer** types on the fly — easy but error-prone (e.g., guessing a number as text). **Explicit** mapping defines types up front — recommended for production for correctness, performance, and predictable behavior.

**Q: Can you change a field's mapping after creating an index?**
No — you **cannot change the type of an existing field**. You must **create a new index** with the corrected mapping and **reindex** the data (often using an **alias** to switch seamlessly).

**Q: `object` vs `nested` type?**
An **`object`** field flattens arrays of objects, which can break cross-field matching within an element. A **`nested`** type indexes each object in the array as a **separate hidden document**, preserving the relationship so you can query fields **within the same array element** correctly.

---

## 5. Analysis: Analyzers, Tokenizers, Filters

### Theory

**Analysis** is the process of converting text into **terms** for the inverted index. An **analyzer** has three parts, applied in order:

1. **Character filters** — preprocess raw text (e.g., strip HTML).
2. **Tokenizer** — splits text into **tokens** (e.g., on whitespace/punctuation). Exactly one per analyzer.
3. **Token filters** — modify tokens (lowercase, stop-word removal, **stemming**, synonyms, etc.).

**Built-in analyzers:** `standard` (default), `simple`, `whitespace`, `keyword` (no tokenizing — whole string as one term), `english` (language-aware stemming).

**Key rule:** the **same analyzer is applied at index time and search time** so terms match. `text` fields are analyzed; `keyword` fields are **not** (stored verbatim).

### Program

```json
// Test an analyzer
POST /_analyze
{ "analyzer": "standard", "text": "The QUICK Brown-Foxes!" }
// -> tokens: [the, quick, brown, foxes]
```

### Interview Q&A

**Q: What is an analyzer?**
A component that **tokenizes and normalizes text** into the terms stored in the inverted index. It runs **character filters → tokenizer → token filters** (e.g., lowercase, remove stop words, stem). The same analyzer applies at index and query time.

**Q: What's the difference between a tokenizer and a token filter?**
The **tokenizer** splits text into tokens (e.g., by whitespace/punctuation) — one per analyzer. **Token filters** then transform those tokens (lowercase, stemming, stop-word removal, synonyms). Order: tokenizer first, then filters.

**Q: What is stemming?**
Reducing words to their **root form** (e.g., "running", "ran" → "run") so different inflections of a word **match** each other in search. Done by a token filter / language analyzer.

**Q: Why must index-time and search-time analysis match?**
Because search compares the **analyzed query terms** against the **analyzed indexed terms**. If they differ (e.g., one lowercased and the other not), matches fail. By default Elasticsearch uses the field's analyzer for both.

---

## 6. Indexing Documents (CRUD)

### Theory

Documents are created/read/updated/deleted via the REST API. Writes are **near-real-time** — a document is searchable after a **refresh** (default every 1 second), not instantly.

### Program

```json
// CREATE / INDEX (auto-generated _id with POST, explicit with PUT)
POST /products/_doc
{ "name": "Laptop", "price": 999 }

PUT /products/_doc/1
{ "name": "Phone", "price": 599 }

// READ
GET /products/_doc/1

// UPDATE (partial)
POST /products/_update/1
{ "doc": { "price": 549 } }

// DELETE
DELETE /products/_doc/1

// BULK (efficient multi-op)
POST /_bulk
{ "index": { "_index": "products", "_id": "2" } }
{ "name": "Tablet", "price": 399 }
{ "index": { "_index": "products", "_id": "3" } }
{ "name": "Watch", "price": 199 }
```

### Interview Q&A

**Q: How do you efficiently index many documents?**
Use the **Bulk API** (`_bulk`) — it batches many index/update/delete operations in a single request, dramatically reducing overhead versus one call per document.

**Q: Are documents immediately searchable after indexing?**
Not instantly — Elasticsearch is **near-real-time**. A document becomes searchable after a **refresh** (default **every 1 second**), when the in-memory buffer is written to a new searchable segment.

**Q: How do updates work internally?**
Documents are **immutable** in Lucene. An "update" actually **marks the old document as deleted and indexes a new version** (with an incremented `_version`). Deleted docs are purged later during **segment merging**.

**Q: What is optimistic concurrency control in Elasticsearch?**
Each document has a **`_seq_no` and `_primary_term`** (older: `_version`). On update you can require these to match, so **concurrent updates** don't silently overwrite each other — a conflicting write fails and you retry.

---

## 7. Query DSL: Full-Text vs Term-Level

### Theory

Elasticsearch queries use a JSON **Query DSL**. Two big families:

- **Full-text queries** — **analyze** the query string (same analyzer as the field) before matching; used on **`text`** fields. Examples: `match`, `match_phrase`, `multi_match`, `query_string`.
- **Term-level queries** — **do NOT analyze** the query; match the **exact term** in the inverted index; used on **structured/`keyword`** fields. Examples: `term`, `terms`, `range`, `exists`, `prefix`, `wildcard`, `fuzzy`.

**Classic gotcha:** a `term` query on a `text` field often fails because the text was analyzed (lowercased/tokenized) but your term wasn't. Use `match` on text, `term` on keyword.

### Program

```json
// Full-text: analyzes "quick fox" -> matches docs containing those terms
GET /articles/_search
{ "query": { "match": { "body": "quick fox" } } }

// Phrase match (order matters)
GET /articles/_search
{ "query": { "match_phrase": { "body": "quick brown fox" } } }

// Term-level: exact match on keyword
GET /products/_search
{ "query": { "term": { "sku": "ABC-123" } } }

// Range (numbers/dates)
GET /products/_search
{ "query": { "range": { "price": { "gte": 100, "lte": 500 } } } }

// Date range
GET /logs/_search
{ "query": { "range": { "timestamp": { "gte": "now-7d/d", "lte": "now" } } } }
```

### Interview Q&A

**Q: Full-text vs term-level queries?**
**Full-text queries** (`match`, `multi_match`) **analyze** the query string before matching — used on **`text`** fields like an article body. **Term-level queries** (`term`, `range`, `exists`) match the **exact** stored term **without analysis** — used on **structured/`keyword`** fields like ids, statuses, numbers, dates.

**Q: `match` vs `match_phrase`?**
`match` finds documents containing **any/all of the analyzed terms** (order-independent). `match_phrase` requires the terms to appear in the **same order and adjacency** — used for exact phrase search.

**Q: Why does a `term` query on a text field return nothing?**
Because the `text` field was **analyzed** at index time (e.g., lowercased/tokenized) but `term` matches the **raw, un-analyzed** value. The indexed term ("laptop") doesn't equal your term ("Laptop"). Use `match` on text, or query the field's `.keyword` sub-field.

**Q: How do you do a date-range search?**
A **`range`** query on a date field with `gte`/`lte`, supporting **date math** like `now-7d/d` (7 days ago, rounded to the day).

---

## 8. Compound Queries & the bool Query

### Theory

The **`bool` query** combines multiple clauses:

- **`must`** — all must match; **contributes to score** (AND).
- **`should`** — optional; matching **boosts the score** (OR-ish). With no `must`, at least one `should` must match (configurable via `minimum_should_match`).
- **`must_not`** — must **not** match; **filter context** (no score).
- **`filter`** — must match but in **filter context** — **no scoring**, **cacheable**, faster.

### Program

```json
GET /products/_search
{
  "query": {
    "bool": {
      "must":     [ { "match": { "name": "wireless headphones" } } ],
      "filter":   [ { "range": { "price": { "lte": 200 } } },
                    { "term":  { "in_stock": true } } ],
      "should":   [ { "match": { "brand": "Sony" } } ],
      "must_not": [ { "term": { "discontinued": true } } ]
    }
  }
}
```

### Interview Q&A

**Q: Explain the bool query clauses.**
**`must`** = required, affects score (AND). **`should`** = optional, boosts score when matched (OR). **`must_not`** = exclusion (filter context, no score). **`filter`** = required but in **filter context** — no scoring, **cached**, faster. Combine them for complex search + filter logic.

**Q: When do you use `filter` vs `must`?**
Use **`filter`** for yes/no conditions where **relevance doesn't matter** (price range, status, dates) — it skips scoring and is **cached**, so it's faster. Use **`must`** when the clause should **influence relevance ranking** (full-text matches).

---

## 9. Relevance Scoring & Query vs Filter Context

### Theory

**Relevance score (`_score`)** ranks how well a document matches a full-text query. Modern Elasticsearch uses **BM25** (an improvement on classic TF-IDF):

- **TF (term frequency)** — more occurrences of the term → higher score.
- **IDF (inverse document frequency)** — rarer terms across the corpus → more weight.
- **Field length norm** — matches in shorter fields score higher.

**Query context** ("how well does this match?") computes a **`_score`**. **Filter context** ("does this match, yes/no?") returns a boolean, **no score**, and results are **cacheable** → faster. `filter` and `must_not` run in filter context.

### Interview Q&A

**Q: How does Elasticsearch rank results?**
By a **relevance `_score`** computed with **BM25**, driven by **term frequency** (more matches = higher), **inverse document frequency** (rarer terms weigh more), and **field-length normalization** (shorter fields score higher). Results are returned in descending score order.

**Q: Query context vs filter context?**
**Query context** asks "**how well** does this match?" and computes a relevance **`_score`**. **Filter context** asks "**does** this match (yes/no)?" — it produces **no score**, and is **cached** for reuse, making it **faster**. Use filters for structured conditions.

**Q: What is BM25?**
The default **relevance ranking algorithm** in modern Elasticsearch — a probabilistic refinement of TF-IDF that handles term frequency saturation and document length better, giving more accurate full-text ranking.

---

## 10. Aggregations

### Theory

**Aggregations** compute **analytics/summaries** over documents (the analytics side of Elasticsearch — like SQL `GROUP BY` + aggregate functions). Types:

- **Metric aggregations** — compute values: `avg`, `sum`, `min`, `max`, `stats`, `cardinality` (distinct count), `value_count`.
- **Bucket aggregations** — group documents into buckets: `terms` (group by field value), `range`, `date_histogram`, `histogram`, `filters`.
- **Pipeline aggregations** — operate on the output of other aggregations (e.g., moving average, cumulative sum, derivative).

Aggregations can be **nested** (bucket → sub-aggregation) for powerful drill-downs.

### Program

```json
// Average price per category, with overall stats
GET /products/_search
{
  "size": 0,                                  // don't return docs, just aggs
  "aggs": {
    "by_category": {
      "terms": { "field": "category", "size": 10 },   // bucket per category
      "aggs": {
        "avg_price": { "avg": { "field": "price" } }   // metric per bucket
      }
    },
    "unique_brands": { "cardinality": { "field": "brand" } }
  }
}

// Logs per day (date histogram) grouped by status
GET /logs/_search
{
  "size": 0,
  "aggs": {
    "per_day": {
      "date_histogram": { "field": "timestamp", "calendar_interval": "day" },
      "aggs": { "by_status": { "terms": { "field": "status" } } }
    }
  }
}
```

### Interview Q&A

**Q: What are aggregations used for?**
To compute **metrics, statistics, and summaries** over data — counts, averages, distinct counts, histograms — the **analytics** side of Elasticsearch. E.g., group logs by status code and count occurrences, or compute average price per category.

**Q: Metric vs bucket aggregation?**
**Metric** aggregations compute a **value** over documents (avg, sum, min, max, cardinality). **Bucket** aggregations **group documents** into buckets (terms, ranges, date histograms). You **nest** metrics inside buckets to compute per-group statistics.

**Q: How do you count distinct values?**
The **`cardinality`** aggregation — it gives an **approximate distinct count** (using HyperLogLog) that's memory-efficient at scale; exact counts are too expensive on big data.

**Q: `size: 0` in an aggregation query — why?**
It tells Elasticsearch to **skip returning matching documents** and return **only the aggregation results**, saving bandwidth and compute when you only want the analytics.

---

## 11. Cluster Architecture & Node Roles

### Theory

A **cluster** is one or more **nodes** working together. Node roles:

- **Master node** — manages cluster state (creating indices, allocating shards, tracking nodes). One is **elected**; others are master-eligible.
- **Data node** — stores shards and handles indexing/search.
- **Coordinating node** — routes requests, scatters to shards, and **gathers/merges** results (every node can coordinate).
- **Ingest node** — runs **ingest pipelines** (pre-process documents before indexing).
- Specialized: ML, transform, etc.

**Cluster health:** **green** (all primaries + replicas assigned), **yellow** (primaries OK, some replicas unassigned), **red** (some primaries unassigned — data missing).

**Split-brain** — avoided by requiring a **quorum** of master-eligible nodes (`minimum_master_nodes` in older versions; automatic in 7.x+).

### Interview Q&A

**Q: What are the main node roles?**
**Master** (manages cluster state/shard allocation), **data** (stores and searches shards), **coordinating** (routes and merges query results), and **ingest** (pre-processing pipelines). Small clusters may combine roles; large clusters dedicate them.

**Q: What do green/yellow/red cluster status mean?**
**Green** = all primary and replica shards assigned. **Yellow** = all primaries assigned but **some replicas unassigned** (still fully functional, less redundancy). **Red** = **some primaries unassigned** — part of the data is unavailable.

**Q: What is split-brain and how is it prevented?**
**Split-brain** is when a network partition causes **two nodes to think they're master**, risking divergent state. It's prevented by requiring a **quorum/majority** of master-eligible nodes to elect a master (handled automatically in 7.x+).

---

## 12. Sharding, Replication & Routing

### Theory

- **Sharding** splits an index into **primary shards** distributed across nodes → **horizontal scale** (data + write throughput).
- **Replication** copies each primary into **replica shards** on other nodes → **HA + read scaling**.
- **Routing** — a document's shard is chosen by `shard = hash(routing) % number_of_primary_shards` (default routing = `_id`). This is why **the number of primary shards is fixed at index creation** (changing it would break routing) — to change it you **reindex**.

**Sizing guidance:** avoid **too many small shards** (overhead) or **too few huge shards** (hard to move/recover); a common target is shards in the ~10–50 GB range.

### Interview Q&A

**Q: Sharding vs replication?**
**Sharding** splits data across primary shards for **scalability** (handle more data and writes). **Replication** copies shards for **availability and read throughput** (survive node failures, serve more reads). They solve different problems and are used together.

**Q: Can you change the number of primary shards after creating an index?**
No — the **primary shard count is fixed at creation** because document **routing** depends on it (`hash(_id) % num_primaries`). To change it you **reindex** into a new index (or use the **split/shrink** APIs). You **can** change the number of **replicas** dynamically.

**Q: How does Elasticsearch decide which shard a document goes to?**
By **routing**: `shard = hash(routing_value) % number_of_primary_shards`, where the routing value defaults to the document **`_id`**. This deterministically and evenly distributes documents across primaries.

**Q: What problems come from too many shards?**
**Oversharding** — each shard has memory/file-handle/overhead costs, so too many small shards waste resources and slow the cluster. Aim for reasonably sized shards and use **ILM/rollover** for time-series data instead of one giant or thousands of tiny indices.

---

## 13. Read & Write Path / Near Real-Time

### Theory

**Write path:** a document is sent to the **primary** shard, written to an **in-memory buffer + translog** (transaction log for durability), then periodically a **refresh** flushes the buffer into a new **segment** (making it searchable). Replicas receive the operation in parallel. A **flush** commits segments to disk and clears the translog. **Segment merging** combines small immutable segments and purges deleted docs.

**Read/search path:** a **coordinating node** scatters the query to the relevant shards (**query phase** — each shard returns top matching ids+scores), merges/sorts them, then **fetch phase** retrieves the actual documents from the chosen shards.

**Near-real-time:** because searchability depends on **refresh** (default 1s), there's a small delay between indexing and a document being searchable.

### Interview Q&A

**Q: Why is Elasticsearch "near real-time"?**
Because indexed documents only become searchable after a **refresh** (default **every 1 second**) writes the in-memory buffer into a searchable segment. So there's a ~1s delay — not instant, but near-real-time.

**Q: What is the translog?**
The **transaction log** — every write is appended to it for **durability** before a flush commits segments to disk. If a node crashes, the translog is **replayed** to recover un-flushed operations.

**Q: What are segments and why does merging happen?**
Lucene stores data in immutable **segments**. New writes create new small segments; **merging** periodically combines them into larger ones and **physically removes deleted/updated documents**, improving search performance and reclaiming space.

**Q: Explain the query then fetch process.**
In the **query phase**, the coordinating node sends the query to all relevant shards; each returns the **ids and scores** of its top hits. The coordinator **merges and sorts** them globally, then in the **fetch phase** retrieves the **full documents** for the final top-N from their shards.

---

## 14. Text vs Keyword & Common Pitfalls

### Theory

The single most important field-type decision:

- **`text`** — **analyzed** (tokenized, lowercased, stemmed). Use for **full-text search** (article body, description). **Not** good for sorting/aggregations/exact match.
- **`keyword`** — **not analyzed**; stored as a single exact term. Use for **exact match, sorting, aggregations, filtering** (ids, status, tags, email).

A common mapping is a **multi-field**: index a string as both `text` (for search) and `text.keyword` (for aggregations/sorting).

### Program

```json
PUT /docs
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "fields": { "keyword": { "type": "keyword" } }  // multi-field
      }
    }
  }
}
// Search:    match on title
// Aggregate: terms on title.keyword
```

### Interview Q&A

**Q: `text` vs `keyword` — when to use each?**
Use **`text`** for **full-text search** — it's analyzed so partial/relevance matching works. Use **`keyword`** for **exact matching, filtering, sorting, and aggregations** — it's stored verbatim. Picking wrong is the most common Elasticsearch bug (e.g., can't aggregate on a `text` field, or `term` fails on it).

**Q: What is a multi-field?**
Indexing one source field as **multiple types** — e.g., `title` as `text` (for search) and `title.keyword` as `keyword` (for sorting/aggregations) — so you get both behaviors without duplicating the source.

**Q: Can you aggregate or sort on a `text` field?**
Not by default — `text` fields don't have **doc values** enabled (they'd be too expensive). You aggregate/sort on the **`keyword`** version (often via a multi-field like `field.keyword`).

---

## 15. Performance & Optimization

### Theory

Key levers:

1. **Explicit mappings** — avoid dynamic mapping guessing; use `keyword` vs `text` correctly.
2. **Use `filter` context** for structured conditions (cached, no scoring).
3. **Right-size shards** — avoid over/under-sharding; use **ILM + rollover** for time-series.
4. **Bulk API** for indexing; tune **refresh interval** (increase or disable during heavy bulk loads).
5. **Limit returned fields** (`_source` filtering) and use **pagination wisely** — avoid deep `from/size`; use **`search_after`** or **scroll/PIT** for large result sets.
6. **`cardinality`/approximate aggs** instead of exact where possible.
7. **Index only what you search**; disable indexing on fields you don't query (`"index": false`).
8. Avoid expensive **wildcard/regex/leading-wildcard** queries.

### Interview Q&A

**Q: How do you speed up indexing of a large dataset?**
Use the **Bulk API**, **increase or temporarily disable the refresh interval** during the load (then re-enable), reduce replicas to 0 during initial load and restore after, and use **explicit mappings** so Elasticsearch doesn't re-guess types.

**Q: Why avoid deep pagination (`from`/`size`)?**
Each shard must collect `from + size` results and the coordinator merges them, so deep offsets get **expensive and memory-heavy**. Use **`search_after`** (cursor-based) or **Point-in-Time (PIT)** for large/scrolling result sets instead.

**Q: How do you handle time-series data (logs) efficiently?**
Use **Index Lifecycle Management (ILM)** with **rollover** — write to a rolling alias, create new indices by size/age, move older ones to cheaper tiers (warm/cold/frozen), and delete after retention. This keeps shard sizes healthy and storage costs down.

**Q: How would you make a slow search faster?**
Move structured conditions to **filter context** (cached), ensure correct **`text`/`keyword`** mappings, avoid **leading-wildcard/regex** queries, limit returned **`_source`** fields, and right-size shards. Diagnose with the **Profile API** / `_search?profile=true`.

---

## 16. The ELK / Elastic Stack

### Theory

The **ELK / Elastic Stack** is the popular logging/observability ecosystem:

- **Elasticsearch** — storage, search, and analytics.
- **Logstash** — server-side **ingestion & transformation** pipeline (parse, enrich, filter logs).
- **Kibana** — **visualization & dashboards** UI (also Dev Tools for queries).
- **Beats** — lightweight **data shippers** on edge hosts (Filebeat for logs, Metricbeat for metrics).
- **Ingest pipelines** — lightweight in-Elasticsearch processing (often replacing Logstash for simple cases).

**Typical flow:** Beats/Logstash → Elasticsearch → Kibana.

### Interview Q&A

**Q: What is the ELK stack?**
**Elasticsearch + Logstash + Kibana** (plus **Beats**) — a stack for **collecting, storing, searching, and visualizing** data, especially **logs and metrics**. Beats ship data, Logstash transforms it, Elasticsearch stores/searches, Kibana visualizes.

**Q: Logstash vs Beats vs ingest pipelines?**
**Beats** are lightweight shippers on hosts (minimal processing). **Logstash** is a heavier, flexible ETL pipeline (parsing, enrichment, many inputs/outputs). **Ingest pipelines** run inside Elasticsearch for simple transformations — often enough to skip Logstash.

**Q: What is Kibana used for?**
**Visualization, dashboards, and exploration** of Elasticsearch data, plus operational tools (Dev Tools console, index management, alerting). It's the UI layer of the stack.

---

## 17. Common Query Programs

*(Assume a `products` index with `name (text), category (keyword), price (double), brand (keyword), in_stock (boolean)`.)*

### 1. Full-text search

```json
GET /products/_search
{ "query": { "match": { "name": "wireless mouse" } } }
```

### 2. Exact match + range filter (bool)

```json
GET /products/_search
{
  "query": { "bool": {
    "filter": [ { "term": { "category": "electronics" } },
                { "range": { "price": { "lte": 100 } } } ] } }
}
```

### 3. Multi-field search

```json
GET /products/_search
{ "query": { "multi_match": { "query": "gaming", "fields": ["name", "description"] } } }
```

### 4. Sort by price, return selected fields

```json
GET /products/_search
{
  "_source": ["name", "price"],
  "sort": [ { "price": "desc" } ],
  "query": { "match_all": {} }
}
```

### 5. Average price per category (aggregation)

```json
GET /products/_search
{
  "size": 0,
  "aggs": { "by_cat": { "terms": { "field": "category" },
            "aggs": { "avg_price": { "avg": { "field": "price" } } } } }
}
```

### 6. Distinct brand count

```json
GET /products/_search
{ "size": 0, "aggs": { "brands": { "cardinality": { "field": "brand" } } } }
```

### 7. Date-range log query (last 24h)

```json
GET /logs/_search
{ "query": { "range": { "@timestamp": { "gte": "now-24h", "lte": "now" } } } }
```

### 8. Fuzzy / typo-tolerant search

```json
GET /products/_search
{ "query": { "fuzzy": { "name": { "value": "labtop", "fuzziness": "AUTO" } } } }
```

### 9. Autocomplete (prefix)

```json
GET /products/_search
{ "query": { "match_phrase_prefix": { "name": "wireless he" } } }
```

### 10. Bulk index

```json
POST /_bulk
{ "index": { "_index": "products", "_id": "10" } }
{ "name": "Keyboard", "category": "electronics", "price": 49 }
{ "index": { "_index": "products", "_id": "11" } }
{ "name": "Mouse", "category": "electronics", "price": 25 }
```

---

## 18. Tricky / Gotcha Questions

**1. `term` on a `text` field fails** — text is analyzed (lowercased/tokenized); use `match`, or query `field.keyword`.

**2. Can't aggregate/sort on `text`** — no doc values; use a `keyword` multi-field.

**3. Primary shard count is fixed** at index creation (routing depends on it) — reindex to change it; replicas are dynamic.

**4. Near-real-time, not real-time** — docs are searchable after the ~1s refresh, not instantly.

**5. Documents are immutable** — an update = delete + re-index a new version; space reclaimed on merge.

**6. Mapping types removed in 7.x** — one type per index now.

**7. Deep pagination is expensive** — use `search_after`/PIT, not large `from`.

**8. `object` vs `nested`** — arrays of objects need `nested` type to query within the same element correctly.

**9. Yellow status is OK in single-node** — replicas can't be allocated (no second node), but data is fine.

**10. `filter` is cached and unscored; `must` is scored** — use filter for structured conditions to be faster.

**11. Dynamic mapping can mis-type fields** (e.g., date as text) — prefer explicit mappings in production.

**12. You can't change a field's type in place** — create a new index and reindex (use aliases to switch).

---

## 19. Rapid-Fire One-Liners

- **Elasticsearch** = distributed search/analytics engine on **Lucene**, stores JSON docs.
- **Inverted index** = term → documents; why search is fast.
- **Index** = collection of documents; **document** = JSON record; **shard** = a Lucene index.
- **Primary** shard = data + writes; **replica** = copy for HA + read scaling.
- **Mapping** = schema; **dynamic** (auto) vs **explicit** (defined).
- **Analyzer** = char filters → tokenizer → token filters (produces terms).
- **Stemming** = reduce words to root so inflections match.
- **`text`** = analyzed (full-text search); **`keyword`** = exact (filter/sort/agg).
- **Full-text** queries (`match`) analyze; **term-level** (`term`, `range`) don't.
- **`match`** = any/all terms; **`match_phrase`** = ordered phrase.
- **`bool`**: must (AND/score), should (OR/boost), must_not (exclude), filter (no score, cached).
- **Query context** = scored; **filter context** = boolean, cached, faster.
- **BM25** = default relevance algorithm (TF, IDF, length norm).
- **Aggregations**: metric (avg/sum/cardinality) + bucket (terms/range/date_histogram) + pipeline.
- **`cardinality`** = approximate distinct count.
- **Routing**: shard = hash(_id) % num_primary_shards — primaries fixed at creation.
- **Near-real-time** = searchable after ~1s refresh; **translog** = durability.
- **Segments** immutable; **merge** purges deleted docs.
- **Cluster health**: green/yellow/red.
- **Node roles**: master, data, coordinating, ingest.
- **ELK** = Elasticsearch + Logstash + Kibana (+ Beats).
- **Bulk API** for fast indexing; **search_after/PIT** for deep pagination; **ILM** for logs.

---

## 20. Last-Minute Checklist

The hour before:

- [ ] What Elasticsearch is and that it's built on **Lucene**.
- [ ] **Inverted index** — what it is and why search is fast.
- [ ] Index / document / shard / replica; primary vs replica.
- [ ] **Mapping**: dynamic vs explicit; can't change field types (reindex).
- [ ] **Analyzer** pipeline: char filter → tokenizer → token filter; stemming.
- [ ] **`text` vs `keyword`** (the #1 gotcha) and multi-fields.
- [ ] **Full-text vs term-level** queries; `match` vs `match_phrase`; why `term` fails on text.
- [ ] **`bool` query** clauses; **query vs filter context** (scoring, caching).
- [ ] **Relevance/BM25**: TF, IDF, length norm.
- [ ] **Aggregations**: metric vs bucket vs pipeline; `cardinality`; `size: 0`.
- [ ] **Cluster**: node roles, green/yellow/red, split-brain.
- [ ] **Sharding/replication/routing**; why primary count is fixed.
- [ ] **Near-real-time**, refresh, translog, segments, query-then-fetch.
- [ ] **Performance**: filter context, bulk, search_after, ILM, avoid wildcards.
- [ ] **ELK stack** components.
- [ ] Practice writing a **match + bool filter** query and a **terms + avg aggregation** from memory.

**Interview tips:** Reason from **how data is searched and accessed** — that drives `text` vs `keyword` and full-text vs term-level. For performance, mention **filter context, correct mappings, right-sized shards, and avoiding deep pagination**. Distinguish **sharding (scale) from replication (availability)**. If you've used Elasticsearch with a system of record (e.g., **MongoDB → Elasticsearch via Change Streams** for search), a concrete example like that makes your answers stand out.

---

*Good luck — read it top-to-bottom once, then drill the high-frequency hits: inverted index, `text` vs `keyword`, full-text vs term-level queries, the `bool`/filter-context distinction, and aggregations. Those decide Elasticsearch interviews.*
