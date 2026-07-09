# MongoDB Interview — Last-Minute Prep Guide

> **How to use this:** Skim the **Theory** for crisp definitions, read the **Query/Program** to see it in action, and rehearse the **Interview Q&A** out loud. Bold lines are the one-liners interviewers want to hear. Built for a 2–4 hour final revision.
>
> *Syntax uses MongoDB Shell (mongosh) / Node-style query documents. Concepts apply to MongoDB 5.x–7.x.*

---

## Table of Contents

1. [MongoDB Fundamentals](#1-mongodb-fundamentals)
2. [Documents, BSON & Data Types](#2-documents-bson--data-types)
3. [CRUD Operations](#3-crud-operations)
4. [Query Operators & Projection](#4-query-operators--projection)
5. [Update Operators](#5-update-operators)
6. [Indexing](#6-indexing)
7. [Aggregation Pipeline](#7-aggregation-pipeline)
8. [Schema Design: Embedding vs Referencing](#8-schema-design-embedding-vs-referencing)
9. [Data Modeling Patterns](#9-data-modeling-patterns)
10. [Replication & Replica Sets](#10-replication--replica-sets)
11. [Sharding](#11-sharding)
12. [Transactions & ACID](#12-transactions--acid)
13. [Read/Write Concerns & Consistency](#13-readwrite-concerns--consistency)
14. [Performance & Optimization](#14-performance--optimization)
15. [Aggregation vs map-reduce, Change Streams, GridFS](#15-aggregation-vs-map-reduce-change-streams-gridfs)
16. [Common Coding Programs](#16-common-coding-programs)
17. [Tricky / Gotcha Questions](#17-tricky--gotcha-questions)
18. [Rapid-Fire One-Liners](#18-rapid-fire-one-liners)
19. [Last-Minute Checklist](#19-last-minute-checklist)

---

## 1. MongoDB Fundamentals

### Theory

**MongoDB** is an open-source, **document-oriented NoSQL database**. Instead of tables/rows, it stores **documents** (JSON-like) in **collections**.

**Relational → MongoDB mapping:**

| RDBMS | MongoDB |
|---|---|
| Database | Database |
| Table | **Collection** |
| Row | **Document** |
| Column | **Field** |
| Join | **$lookup** / embedding |
| Primary key | **`_id`** field |

**Key properties:** **schema-flexible** (documents in a collection can differ), **horizontally scalable** (sharding), **highly available** (replica sets), stores data as **BSON** (Binary JSON).

**Why/when MongoDB:** flexible/evolving schemas, semi-structured or hierarchical data, high write throughput, rapid development, and horizontal scale. **When NOT:** heavy multi-entity transactions/joins and strict relational integrity — a relational DB may fit better.

### Interview Q&A

**Q: What is MongoDB and how is it different from a relational database?**
MongoDB is a **document-oriented NoSQL** database storing **BSON documents** in **collections** with **flexible schemas**, versus RDBMS's fixed tables/rows/columns. It favors **embedding/denormalization** over joins, scales **horizontally via sharding**, and is ideal for semi-structured, evolving data.

**Q: What is a collection and a document?**
A **document** is a JSON-like set of key-value pairs (the basic unit of data). A **collection** is a group of documents (analogous to a table) — but **schema-less**, so documents can have different fields.

**Q: When would you NOT use MongoDB?**
When the data is **highly relational** with complex multi-table joins and strict integrity/transactional needs across many entities — a relational database is often a better fit there.

**Q: Is MongoDB schema-less?**
It's **schema-flexible**, not truly schema-less — documents in a collection can vary, but you typically design a consistent schema and can enforce it with **schema validation** rules.

---

## 2. Documents, BSON & Data Types

### Theory

- **BSON (Binary JSON)** — MongoDB's binary-encoded storage format. Supports more types than JSON (e.g., `Date`, `ObjectId`, `Decimal128`, binary), is **faster to traverse**, and is **lightweight** for storage/network.
- **`_id`** — every document has a unique **`_id`** (primary key). If not supplied, MongoDB generates an **`ObjectId`** — a 12-byte value (4-byte timestamp + 5-byte random + 3-byte counter) that's roughly time-ordered and globally unique.
- **Common types:** String, Number (int/long/double), `Decimal128`, Boolean, `Date`, `ObjectId`, Array, embedded Document (object), Null, Binary.

### Program

```javascript
{
  _id: ObjectId("64f1a2..."),
  name: "Tejas",
  age: 26,
  skills: ["Python", "MongoDB"],         // array
  address: { city: "Pune", zip: "411001" }, // embedded document
  active: true,
  joined: ISODate("2024-01-15")
}
```

### Interview Q&A

**Q: What is BSON and why use it over JSON?**
**BSON** is MongoDB's **binary JSON** format. It supports **richer data types** (Date, ObjectId, Decimal128, binary), is **faster to parse/traverse**, and is **space-efficient** — better suited for storage and indexing than plain text JSON.

**Q: What is `_id` and ObjectId?**
`_id` is the mandatory **unique primary key** of every document. If you don't provide one, MongoDB auto-generates an **ObjectId** — a 12-byte, time-ordered, globally unique identifier (timestamp + random + counter), so it's roughly sortable by creation time.

**Q: Can `_id` be a custom value?**
Yes — you can set `_id` to any unique value (e.g., an email or natural key), as long as it's unique within the collection. It just must be present and unique.

---

## 3. CRUD Operations

### Theory

**Create, Read, Update, Delete** — the core operations.

### Program

```javascript
// CREATE
db.users.insertOne({ name: "Asha", age: 30 })
db.users.insertMany([{ name: "Ben" }, { name: "Cy" }])

// READ
db.users.find({ age: { $gt: 25 } })          // all matching
db.users.findOne({ name: "Asha" })           // first match
db.users.find().sort({ age: -1 }).limit(5)   // sort desc, top 5
db.users.countDocuments({ age: { $gt: 25 } })

// UPDATE
db.users.updateOne({ name: "Asha" }, { $set: { age: 31 } })
db.users.updateMany({ active: false }, { $set: { active: true } })
db.users.replaceOne({ name: "Ben" }, { name: "Ben", age: 40 })

// UPSERT (insert if not found)
db.users.updateOne({ name: "Dee" }, { $set: { age: 22 } }, { upsert: true })

// DELETE
db.users.deleteOne({ name: "Cy" })
db.users.deleteMany({ active: false })
```

### Interview Q&A

**Q: insertOne vs insertMany; updateOne vs updateMany?**
The `*One` variants affect a **single** (first matching) document; the `*Many` variants affect **all** matching documents. `insertMany` inserts a batch.

**Q: What is an upsert?**
An update that **inserts a new document if no match is found**, otherwise updates the existing one — enabled with `{ upsert: true }`. Useful for idempotent writes.

**Q: `find()` vs `findOne()`?**
`find()` returns a **cursor** over all matching documents; `findOne()` returns a **single** document (the first match) or null.

**Q: How do you paginate results?**
`find().sort(...).skip(n).limit(m)`. For large offsets, **range-based pagination** (e.g., `_id > lastSeenId`) is more efficient than `skip`, which scans skipped documents.

---

## 4. Query Operators & Projection

### Theory

**Comparison:** `$eq`, `$ne`, `$gt`, `$gte`, `$lt`, `$lte`, `$in`, `$nin`.
**Logical:** `$and`, `$or`, `$not`, `$nor`.
**Element:** `$exists`, `$type`.
**Array:** `$all`, `$elemMatch`, `$size`.
**Evaluation:** `$regex`, `$expr`, `$mod`.

**Projection** — choose which fields to return: `1` includes, `0` excludes (can't mix except `_id`).

### Program

```javascript
// Operators
db.products.find({ price: { $gte: 100, $lte: 500 } })
db.products.find({ category: { $in: ["A", "B"] } })
db.products.find({ $or: [{ stock: 0 }, { discontinued: true }] })
db.products.find({ tags: { $all: ["new", "sale"] } })
db.orders.find({ items: { $elemMatch: { qty: { $gt: 5 }, price: { $lt: 50 } } } })

// Projection: return only name & price, hide _id
db.products.find({ price: { $gt: 100 } }, { name: 1, price: 1, _id: 0 })
```

### Interview Q&A

**Q: What is projection?**
Specifying **which fields to return** from matching documents — `1` to include, `0` to exclude. It reduces network/payload and can help **covered queries**. You can't mix include/exclude except for `_id`.

**Q: `$in` vs `$elemMatch`?**
`$in` matches a field against **any value in a list**. `$elemMatch` matches **array elements** that satisfy **multiple conditions simultaneously** within a single element.

**Q: How do you query inside an embedded document/array?**
Use **dot notation** for nested fields (`"address.city": "Pune"`) and `$elemMatch`/positional operators for arrays of objects.

---

## 5. Update Operators

### Theory

**Field:** `$set`, `$unset`, `$inc`, `$mul`, `$rename`, `$min`, `$max`, `$setOnInsert`, `$currentDate`.
**Array:** `$push`, `$pull`, `$addToSet`, `$pop`, `$each`, `$position`, positional `$` and `$[]`.

### Program

```javascript
db.users.updateOne({ _id: 1 }, { $inc: { loginCount: 1 } })       // increment
db.users.updateOne({ _id: 1 }, { $push: { tags: "premium" } })    // append to array
db.users.updateOne({ _id: 1 }, { $addToSet: { tags: "vip" } })    // append if absent
db.users.updateOne({ _id: 1 }, { $pull: { tags: "trial" } })      // remove value
db.users.updateOne({ _id: 1 }, { $unset: { tempField: "" } })     // remove field

// Update a matched array element (positional $)
db.orders.updateOne(
  { _id: 1, "items.sku": "A1" },
  { $set: { "items.$.qty": 10 } }
)
```

### Interview Q&A

**Q: `$push` vs `$addToSet`?**
`$push` **always appends** a value to an array (allows duplicates). `$addToSet` appends **only if the value isn't already present** (keeps unique elements).

**Q: How do you update a specific element in an array?**
Use the **positional operator `$`** (updates the first matching element) or **`$[]`** / **`$[<identifier>]`** filtered positional operators to update all or specific matching elements.

**Q: What does `$setOnInsert` do?**
In an **upsert**, it sets fields **only when a new document is inserted**, not on update — useful for initializing creation-time fields.

---

## 6. Indexing

### Theory

An **index** is a data structure (a **B-tree**) that speeds up reads by avoiding a **full collection scan (COLLSCAN)**. Trade-off: indexes **speed reads but slow writes** and use storage.

**Index types:**

- **Single-field** — on one field.
- **Compound** — on multiple fields; **order matters** (follows the **ESR rule**: Equality, Sort, Range) and the **left-prefix** rule.
- **Multikey** — automatically on **array** fields (indexes each element).
- **Text** — full-text search on string content.
- **Geospatial** (`2d`, `2dsphere`) — location queries.
- **Hashed** — for hashed sharding / even distribution.
- **Unique**, **partial**, **sparse**, **TTL** (auto-expire documents after a time).

**Covered query** — a query satisfied **entirely from the index** (all needed fields are in the index), so MongoDB never reads the documents — very fast.

**`explain()`** — shows the query plan; look for **IXSCAN** (good, index used) vs **COLLSCAN** (bad, full scan).

### Program

```javascript
db.users.createIndex({ email: 1 }, { unique: true })       // unique single-field
db.orders.createIndex({ customerId: 1, orderDate: -1 })    // compound
db.articles.createIndex({ content: "text" })               // text index
db.sessions.createIndex({ createdAt: 1 }, { expireAfterSeconds: 3600 }) // TTL

db.users.find({ email: "a@x.com" }).explain("executionStats")  // check IXSCAN
```

### Interview Q&A

**Q: Why use indexes and what's the trade-off?**
Indexes let MongoDB **locate documents without scanning the whole collection**, dramatically speeding reads. The cost is **slower writes** (each index updates on insert/update) and **extra storage**. Index based on your **query patterns**.

**Q: What is a compound index and why does field order matter?**
An index on **multiple fields**. Order matters because of the **left-prefix rule** — a `{a:1, b:1}` index supports queries on `a` and on `a+b`, but **not on `b` alone**. The **ESR rule** (Equality → Sort → Range) guides optimal ordering.

**Q: What is a multikey index?**
An index on an **array field** — MongoDB indexes **each array element**, enabling efficient queries into arrays. Created automatically when you index an array field.

**Q: What is a covered query?**
A query where **all requested fields are in the index**, so MongoDB returns results **directly from the index** without fetching documents — extremely fast. Requires projection to exclude `_id` (or include it in the index).

**Q: What is a TTL index?**
A special **single-field index on a date** with `expireAfterSeconds` — MongoDB **automatically deletes** documents after that time. Great for sessions, logs, and caches.

**Q: How do you know if an index is being used?**
Run **`.explain("executionStats")`** and check the plan — **IXSCAN** means an index is used; **COLLSCAN** means a full scan (usually a problem). Also check `totalDocsExamined` vs `nReturned`.

---

## 7. Aggregation Pipeline

### Theory

The **aggregation pipeline** processes documents through a sequence of **stages**, each transforming the stream and passing output to the next — like an assembly line.

**Common stages:**

- **`$match`** — filter documents (like WHERE). Put it **early** to reduce data and use indexes.
- **`$group`** — group by a key and compute aggregates (`$sum`, `$avg`, `$min`, `$max`, `$push`, `$addToSet`, `$count`).
- **`$project`** — reshape/select fields, compute new ones.
- **`$sort`**, **`$limit`**, **`$skip`** — order and paginate.
- **`$lookup`** — **left outer join** to another collection.
- **`$unwind`** — deconstruct an array into one document per element.
- **`$addFields`/`$set`**, **`$count`**, **`$facet`**, **`$bucket`**, **`$out`/`$merge`** (write results).

### Program

```javascript
// Total sales per customer, top 5, joined with customer info
db.orders.aggregate([
  { $match: { status: "completed" } },                       // filter
  { $group: { _id: "$customerId", total: { $sum: "$amount" }, cnt: { $sum: 1 } } },
  { $sort: { total: -1 } },                                  // highest first
  { $limit: 5 },
  { $lookup: {                                               // join customers
      from: "customers", localField: "_id",
      foreignField: "_id", as: "customer" } },
  { $unwind: "$customer" },
  { $project: { _id: 0, name: "$customer.name", total: 1, cnt: 1 } }
])
```

### Interview Q&A

**Q: What is the aggregation pipeline?**
A **multi-stage data-processing framework** where documents flow through stages (`$match`, `$group`, `$sort`, `$lookup`, etc.), each transforming the stream and passing results onward — used for analytics, reshaping, and joins.

**Q: `$match` placement — why does it matter?**
Put `$match` (and `$sort` on indexed fields) **as early as possible** so it can **use indexes** and **reduce the document volume** flowing through later stages — a key optimization.

**Q: What does `$lookup` do?**
Performs a **left outer join** to another collection in the same database, embedding matching documents as an array — MongoDB's way of joining without a separate query.

**Q: `$group` — how does it work?**
Groups documents by the `_id` expression and computes **accumulators** (`$sum`, `$avg`, `$max`, `$push`, etc.) per group — like SQL `GROUP BY` with aggregates.

**Q: What does `$unwind` do?**
**Flattens an array field**, outputting **one document per array element** — often used before `$group` to aggregate over array contents.

**Q: Aggregation pipeline vs find()?**
`find()` retrieves and filters documents. The **aggregation pipeline** does **multi-stage transformations, grouping, joins, and computed fields** — far more powerful for analytics.

---

## 8. Schema Design: Embedding vs Referencing

### Theory

The central MongoDB design decision: **embed** related data in one document, or **reference** it across documents.

- **Embedding (denormalization)** — store related data **inside** the parent document. **Pros:** single read (no join), atomic updates, fast. **Cons:** documents can grow large (16 MB limit), duplication, harder to update shared data.
- **Referencing (normalization)** — store related data in **separate** documents and link by `_id` (resolve via `$lookup` or a second query). **Pros:** no duplication, smaller documents, good for shared/large data. **Cons:** needs joins/extra queries.

**Rules of thumb:**
- **Embed** for **one-to-few**, data **read together**, and that **changes together** ("contains"/"has-a").
- **Reference** for **one-to-many/many-to-many**, **large** or **shared** data, or data that **grows unboundedly**.

**16 MB** is the maximum BSON document size — a hard constraint on embedding.

### Program

```javascript
// EMBEDDED: order with its line items (read together)
{ _id: 1, customer: "Asha",
  items: [ { sku: "A1", qty: 2 }, { sku: "B2", qty: 1 } ] }

// REFERENCED: post references author by _id
{ _id: 101, title: "Mongo tips", authorId: ObjectId("...") }
// resolve with $lookup or db.authors.findOne({_id: authorId})
```

### Interview Q&A

**Q: Embedding vs referencing — how do you decide?**
**Embed** when related data is **read together, owned by the parent, one-to-few, and changes together** — it gives single-read performance and atomic updates. **Reference** when data is **shared, large, one-to-many/many-to-many, or grows unboundedly** — to avoid duplication and the 16 MB limit. The decision is driven by **access patterns** and **cardinality**.

**Q: What's the document size limit and why does it matter?**
**16 MB** per BSON document. It caps how much you can embed — for unbounded growth (e.g., comments on a viral post), you must **reference** or use the **outlier/bucket pattern** instead of embedding everything.

**Q: How does MongoDB handle a "join"?**
Either by **embedding** related data (no join needed) or with **`$lookup`** in the aggregation pipeline (a left outer join). MongoDB favors embedding to avoid joins where access patterns allow.

**Q: What's the trade-off of denormalization (embedding)?**
Faster reads and atomic writes, but **data duplication** and **harder updates** when the duplicated data changes — you may have to update many documents.

---

## 9. Data Modeling Patterns

### Theory

Well-known MongoDB schema patterns interviewers like:

- **Bucket pattern** — group time-series/IoT readings into "bucket" documents (e.g., per hour) to reduce document count and index size.
- **Outlier pattern** — handle a few documents with abnormally large arrays by overflowing into separate documents.
- **Computed pattern** — precompute and store aggregates (e.g., totals) to avoid recomputation on read.
- **Subset pattern** — embed only the **frequently-accessed subset** (e.g., last 10 reviews) and reference the rest.
- **Schema versioning pattern** — add a `schemaVersion` field to evolve schemas gracefully.
- **Extended reference** — embed a **few key fields** from a referenced document to avoid joins for common reads.

### Interview Q&A

**Q: How would you model time-series / IoT data?**
Use the **bucket pattern** — group many readings into a single document per time window (e.g., one doc per device per hour with an array of readings), which **reduces document count, index size, and improves read efficiency**. (Modern MongoDB also has native **time-series collections**.)

**Q: How do you avoid expensive joins for common reads?**
Use the **extended reference / subset pattern** — embed the **few fields** you frequently need from the referenced document (e.g., author name on a post) so common reads need no `$lookup`, while keeping the full record referenced.

**Q: How do you evolve a schema in production?**
Use the **schema versioning pattern** (a `schemaVersion` field) and handle multiple versions in the application, migrating documents lazily or in batches — MongoDB's flexible schema makes this incremental.

---

## 10. Replication & Replica Sets

### Theory

A **replica set** is a group of `mongod` processes maintaining the **same data set** for **redundancy and high availability**.

- **Primary** — receives all **writes**; there's exactly one.
- **Secondaries** — replicate the primary's **oplog** (operations log) and can serve **reads** (if allowed).
- **Arbiter** — votes in elections but holds **no data** (used to break ties).
- **Automatic failover** — if the primary goes down, an **election** picks a new primary from the secondaries; the old primary rejoins as a secondary and re-syncs.
- **Oplog** — a capped collection recording all write operations, used by secondaries to stay in sync.

### Interview Q&A

**Q: What is a replica set and why use it?**
A group of MongoDB nodes holding the **same data** — one **primary** (writes) and **secondaries** (replicas). It provides **high availability, redundancy, and automatic failover**, so the database survives node failures.

**Q: How does automatic failover work?**
If the primary becomes unavailable, the remaining nodes hold an **election** and promote a **secondary** to primary. When the old primary recovers, it **rejoins as a secondary** and syncs missed operations via the **oplog**.

**Q: What is the oplog?**
The **operations log** — a special capped collection on the primary that records every write. Secondaries **replay** it to replicate data and stay consistent.

**Q: Can you read from secondaries?**
Yes, by setting **read preference** (e.g., `secondaryPreferred`) — useful for scaling reads, but reads may be **slightly stale** (eventual consistency) due to replication lag.

**Q: What is an arbiter?**
A replica-set member that **participates in elections but stores no data** — used to provide a **voting majority** with minimal resources (e.g., in a 2-data-node setup).

---

## 11. Sharding

### Theory

**Sharding** = **horizontal scaling** by **distributing data across multiple machines (shards)**, letting the database grow beyond a single server's limits.

**Components:**

- **Shard** — a node (usually a replica set) holding a **subset** of the data.
- **mongos** — the **query router** apps connect to; routes queries to the right shard(s).
- **Config servers** — store **cluster metadata** and the shard-key ranges.

**Shard key** — the field(s) MongoDB uses to **partition** data into **chunks** distributed across shards. **Choosing it well is critical** — it should have **high cardinality, low frequency, and non-monotonic** values to spread load evenly.

**Sharding strategies:**
- **Ranged sharding** — chunks by value ranges (good for range queries; risk of hotspots).
- **Hashed sharding** — hashes the key for **even distribution** (good for write scaling; bad for range queries).

### Interview Q&A

**Q: What is sharding and when do you need it?**
**Sharding** distributes data **horizontally across multiple shards** to scale beyond one server's capacity (storage, throughput). You need it when a **single replica set can't handle the data volume or load**.

**Q: What is a shard key and how do you choose one?**
The field(s) that determine how data is **partitioned across shards**. A good shard key has **high cardinality, even value distribution (low frequency), and is non-monotonic** — so data and load spread evenly. A poor key (e.g., monotonically increasing like a timestamp) creates **hotspots**.

**Q: Ranged vs hashed sharding?**
**Ranged** splits by value ranges — efficient for **range queries** but can create **hotspots** if keys are monotonic. **Hashed** distributes by a hash of the key — gives **even write distribution** but **range queries must hit all shards**.

**Q: What do mongos and config servers do?**
**mongos** is the **router** that directs client queries to the appropriate shard(s) and merges results. **Config servers** store the **cluster metadata** (chunk ranges, shard mapping).

**Q: Replication vs sharding?**
**Replication** copies the **same data** across nodes for **availability/redundancy**. **Sharding** splits **different data** across nodes for **scalability**. Production clusters often use **both** (each shard is a replica set).

---

## 12. Transactions & ACID

### Theory

MongoDB provides **ACID guarantees at the single-document level** by default — a write to one document (even with embedded arrays/objects) is **atomic**. This is why embedding can avoid the need for multi-document transactions.

**Multi-document ACID transactions** are supported since **v4.0 (replica sets)** and **v4.2 (sharded clusters)** — for operations spanning multiple documents/collections that must all succeed or fail together.

**Trade-off:** multi-document transactions add **overhead**; good schema design (embedding related data) often **avoids needing them**.

### Program

```javascript
const session = db.getMongo().startSession();
session.startTransaction();
try {
  const accounts = session.getDatabase("bank").accounts;
  accounts.updateOne({ _id: 1 }, { $inc: { balance: -100 } });
  accounts.updateOne({ _id: 2 }, { $inc: { balance:  100 } });
  session.commitTransaction();          // all-or-nothing
} catch (e) {
  session.abortTransaction();
} finally {
  session.endSession();
}
```

### Interview Q&A

**Q: Does MongoDB support ACID transactions?**
Yes. **Single-document** operations are **always atomic** (ACID). **Multi-document ACID transactions** are supported since **v4.0** for replica sets and **v4.2** for sharded clusters.

**Q: Why does good schema design reduce the need for transactions?**
Because **embedding** related data into one document means a single **atomic** write updates everything together — no multi-document transaction needed. You only need transactions when data must change across **separate** documents/collections.

**Q: What's the cost of multi-document transactions?**
They add **performance overhead** and locking, and should be kept **short**. MongoDB's guidance is to **model data to minimize** the need for them.

---

## 13. Read/Write Concerns & Consistency

### Theory

- **Write concern** — the **acknowledgment level** for writes: `w: 1` (primary only), `w: "majority"` (majority of replica set — durable), `w: 0` (fire-and-forget). Higher = safer but slower.
- **Read concern** — the **consistency/isolation** of reads: `local`, `majority` (only majority-committed data), `linearizable`.
- **Read preference** — which node to read from: `primary`, `primaryPreferred`, `secondary`, `secondaryPreferred`, `nearest`.
- **Consistency model:** **strong** when reading from the primary; **eventual** when reading from secondaries (replication lag).

### Interview Q&A

**Q: What is write concern?**
The level of **acknowledgment** MongoDB requires before confirming a write. `w:1` waits for the primary; `w:"majority"` waits for a majority of replicas (durable, survives failover); `w:0` doesn't wait. It's a **durability vs latency** trade-off.

**Q: Strong vs eventual consistency in MongoDB?**
Reads from the **primary** are **strongly consistent** (latest data). Reads from **secondaries** can be **eventually consistent** (slightly stale due to replication lag). You choose via **read preference**.

**Q: How do you ensure a write survives a primary failure?**
Use **`w: "majority"`** write concern — the write is acknowledged only after a **majority** of replica-set members have it, so it persists even if the primary fails and a new one is elected.

---

## 14. Performance & Optimization

### Theory

Key levers:

1. **Index** the fields used in queries, sorts, and joins (avoid COLLSCAN).
2. Use **`explain()`** to verify **IXSCAN** and check `totalDocsExamined`.
3. **Projection** — return only needed fields; aim for **covered queries**.
4. **`$match`/`$sort` early** in aggregation pipelines.
5. Right **schema design** (embed for read-together data) to avoid joins.
6. Avoid large **`skip`** for pagination — use range-based.
7. Watch the **working set** fits in RAM; use the **WiredTiger** cache.
8. Avoid **unbounded array growth** and oversized documents.
9. Use **bulk operations** for many writes.

### Program

```javascript
db.orders.find({ customerId: 5 }).explain("executionStats")
// Check: stage = IXSCAN (good), nReturned vs totalDocsExamined ratio

db.orders.bulkWrite([
  { insertOne: { document: { _id: 1, amt: 10 } } },
  { updateOne: { filter: { _id: 2 }, update: { $inc: { amt: 5 } } } }
])
```

### Interview Q&A

**Q: How do you diagnose and fix a slow MongoDB query?**
Run **`.explain("executionStats")`** — if it shows **COLLSCAN** or `totalDocsExamined` ≫ `nReturned`, the query lacks a good index. Fix by **adding an appropriate index** (single/compound following ESR), adding **projection** for a covered query, and ensuring `$match` uses indexed fields.

**Q: What is the WiredTiger storage engine?**
MongoDB's default storage engine — provides **document-level concurrency**, **compression**, and an in-memory **cache** for the working set. Performance is best when the **working set fits in RAM**.

**Q: How do you efficiently perform many writes?**
Use **`bulkWrite()`** to batch inserts/updates/deletes in one round trip (ordered or unordered), reducing network overhead versus many individual calls.

**Q: Why avoid large `skip` for pagination?**
`skip(n)` still **scans and discards** the first `n` documents, getting slower as `n` grows. **Range-based pagination** (`_id > lastId`) using an index is far more efficient.

---

## 15. Aggregation vs map-reduce, Change Streams, GridFS

### Theory

- **map-reduce** — older, flexible but **slower** data-processing approach using JS functions. **Aggregation pipeline is preferred** for almost all cases (faster, simpler).
- **Change Streams** — subscribe to **real-time data changes** (insert/update/delete) on a collection/db, built on the oplog — used for CDC, syncing to search engines (e.g., Elasticsearch), and event-driven apps.
- **GridFS** — a spec for storing **files larger than 16 MB** by splitting them into chunks across two collections (`fs.files`, `fs.chunks`).

### Interview Q&A

**Q: Aggregation pipeline vs map-reduce?**
The **aggregation pipeline** is **faster, simpler, and recommended** for nearly all aggregation needs (native, optimized stages). **map-reduce** is the older, JavaScript-based approach — more flexible for complex custom logic but slower; rarely needed now.

**Q: What are Change Streams and a use case?**
**Change Streams** let applications **subscribe to real-time changes** (insert/update/delete) using the oplog. A common use case is **CDC** — propagating MongoDB changes to **Elasticsearch** for search, to a cache, or to trigger event-driven workflows.

**Q: What is GridFS?**
A convention for storing and retrieving **files exceeding the 16 MB** document limit by **splitting them into chunks** across `fs.files` (metadata) and `fs.chunks` (data) collections.

---

## 16. Common Coding Programs

*(Assume an `orders` collection with `customerId, amount, status, orderDate, items[]`.)*

### 1. Total amount per customer

```javascript
db.orders.aggregate([
  { $group: { _id: "$customerId", total: { $sum: "$amount" } } }
])
```

### 2. Top 3 customers by spend

```javascript
db.orders.aggregate([
  { $group: { _id: "$customerId", total: { $sum: "$amount" } } },
  { $sort: { total: -1 } },
  { $limit: 3 }
])
```

### 3. Count orders by status

```javascript
db.orders.aggregate([ { $group: { _id: "$status", cnt: { $sum: 1 } } } ])
```

### 4. Orders above average amount

```javascript
const avg = db.orders.aggregate([{ $group: { _id: null, a: { $avg: "$amount" } } }]).toArray()[0].a
db.orders.find({ amount: { $gt: avg } })
```

### 5. Find duplicate emails (users)

```javascript
db.users.aggregate([
  { $group: { _id: "$email", count: { $sum: 1 }, ids: { $push: "$_id" } } },
  { $match: { count: { $gt: 1 } } }
])
```

### 6. Join orders with customers ($lookup)

```javascript
db.orders.aggregate([
  { $lookup: { from: "customers", localField: "customerId",
               foreignField: "_id", as: "cust" } },
  { $unwind: "$cust" },
  { $project: { _id: 0, name: "$cust.name", amount: 1 } }
])
```

### 7. Unwind items and sum quantity per product

```javascript
db.orders.aggregate([
  { $unwind: "$items" },
  { $group: { _id: "$items.sku", totalQty: { $sum: "$items.qty" } } },
  { $sort: { totalQty: -1 } }
])
```

### 8. Monthly sales (group by month)

```javascript
db.orders.aggregate([
  { $group: {
      _id: { $dateToString: { format: "%Y-%m", date: "$orderDate" } },
      total: { $sum: "$amount" } } },
  { $sort: { _id: 1 } }
])
```

### 9. Update: add a tag to all active users

```javascript
db.users.updateMany({ active: true }, { $addToSet: { tags: "active" } })
```

### 10. Create a compound index and verify

```javascript
db.orders.createIndex({ customerId: 1, orderDate: -1 })
db.orders.find({ customerId: 5 }).sort({ orderDate: -1 }).explain("executionStats")
```

---

## 17. Tricky / Gotcha Questions

**1. MongoDB is schema-flexible, not schema-less** — you can (and should) enforce structure with **schema validation**.

**2. Compound index left-prefix rule** — `{a:1,b:1}` helps queries on `a` and `a+b`, but **not `b` alone**.

**3. 16 MB document limit** — caps embedding; use referencing/bucket pattern for unbounded growth.

**4. `skip()` doesn't scale** — it scans skipped docs; use range-based pagination.

**5. Single-document writes are atomic; multi-document needs explicit transactions** (4.0+).

**6. Reads from secondaries can be stale** (eventual consistency) — set read preference deliberately.

**7. `$push` allows duplicates; `$addToSet` doesn't.**

**8. A bad (monotonic) shard key creates hotspots** — avoid timestamps/auto-increment as shard keys.

**9. COLLSCAN in `explain()` = missing index** — almost always a red flag for large collections.

**10. Projection can't mix include and exclude** (except `_id`).

**11. ObjectId is roughly time-sortable** — it embeds a creation timestamp.

**12. `$lookup` is a left outer join** — non-matches still appear (with empty array).

---

## 18. Rapid-Fire One-Liners

- **MongoDB** = document-oriented NoSQL; stores **BSON** in **collections**.
- **Collection** = table; **document** = row; **field** = column; **`_id`** = primary key.
- **BSON** = binary JSON with richer types, faster traversal.
- **ObjectId** = 12-byte, time-ordered unique id (auto-generated).
- **CRUD:** insert/find/update/delete; `*One` vs `*Many`; **upsert** = update-or-insert.
- **Index** avoids COLLSCAN; **compound** order matters (ESR + left-prefix).
- **Multikey** = array index; **TTL** = auto-expire; **text** = full-text; **covered query** = served from index.
- **`explain()`**: IXSCAN good, COLLSCAN bad.
- **Aggregation pipeline** = staged transforms (`$match`, `$group`, `$lookup`, `$unwind`...).
- **`$match` early** to use indexes and cut data.
- **`$lookup`** = left outer join; **`$unwind`** = flatten arrays.
- **Embed** for read-together/one-to-few; **reference** for shared/large/unbounded.
- **16 MB** max document size.
- **Replica set** = HA via primary + secondaries + automatic failover (oplog).
- **Sharding** = horizontal scale; **shard key** must be high-cardinality, non-monotonic.
- **Ranged** sharding for range queries; **hashed** for even distribution.
- **mongos** routes; **config servers** hold metadata.
- **Single-doc writes atomic**; multi-doc transactions since 4.0/4.2.
- **Write concern** `w:majority` = durable; secondaries = eventual consistency.
- **Change Streams** = real-time CDC; **GridFS** = files >16 MB.
- **WiredTiger** = default engine (doc-level concurrency, compression).

---

## 19. Last-Minute Checklist

The hour before:

- [ ] RDBMS→Mongo mapping; what BSON and ObjectId are.
- [ ] CRUD + upsert; `*One` vs `*Many`.
- [ ] Query operators (`$gt`, `$in`, `$or`, `$elemMatch`) and projection.
- [ ] Update operators (`$set`, `$inc`, `$push` vs `$addToSet`, positional `$`).
- [ ] **Indexing**: types, compound left-prefix + ESR, multikey, TTL, covered query, `explain()` (IXSCAN vs COLLSCAN).
- [ ] **Aggregation pipeline**: `$match`, `$group`, `$sort`, `$lookup`, `$unwind`, `$project` — and `$match` early.
- [ ] **Embedding vs referencing** decision rules + 16 MB limit.
- [ ] Data modeling patterns (bucket, subset, extended reference).
- [ ] **Replica sets**: primary/secondary/arbiter, oplog, automatic failover.
- [ ] **Sharding**: shard key choice, ranged vs hashed, mongos/config servers.
- [ ] Transactions: single-doc atomic vs multi-doc (4.0/4.2).
- [ ] Write/read concern; strong vs eventual consistency.
- [ ] Performance: indexing, projection, covered queries, bulkWrite, avoid skip.
- [ ] Change Streams (CDC) and GridFS.
- [ ] Practice: write an **aggregation** (top-N per group, $lookup join, $unwind sum) from memory.

**Interview tips:** For design questions, always reason from **access patterns** — "how is this data read and written?" — then justify **embed vs reference**. For performance, mention **`explain()` and indexes**. For scale, distinguish **replication (availability)** from **sharding (scalability)**. Concrete examples from real projects (e.g., syncing MongoDB to Elasticsearch via Change Streams) make answers stand out.

---

*Good luck — read it top-to-bottom once, then drill the high-frequency hits: indexing (compound/ESR/covered), the aggregation pipeline, embedding vs referencing, and replication vs sharding. Those decide MongoDB interviews.*
