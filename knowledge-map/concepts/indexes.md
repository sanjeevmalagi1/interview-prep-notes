# Indexes (Relational Databases)

## Overview

An index is a separate, ordered data structure that maps column value(s) to
the physical location of the rows that hold them, so the engine can find
matching rows without scanning the whole table. It's the single biggest lever
for read performance in a relational database, and it's a classic tradeoff:
every index speeds up some reads and slows down every write (and costs
storage) — so indexing well is about knowing your query patterns, not adding
indexes reflexively.

**Rule of thumb:** index columns that appear in `WHERE`, `JOIN ON`, `ORDER
BY`, or `GROUP BY` clauses on tables with meaningful row counts and read-heavy
access; don't index write-heavy tables, low-cardinality columns, or tiny
tables just because a query touches them.

---

## How they work

### The core data structure: B-tree (B+tree)

- Both MySQL (InnoDB) and Postgres default to a **B+tree** for standard
  indexes — a balanced, sorted tree structure where every leaf is at the same
  depth, and leaves are linked together for efficient range scans.
- Internal nodes hold routing keys to guide the search; leaf nodes hold the
  actual indexed values plus a pointer to the row (see clustered vs
  non-clustered below). Because the tree is balanced and shallow (typically
  3-4 levels even for millions of rows, since each node holds hundreds of
  entries), a lookup is O(log n) disk/page reads, not O(n).
- B-trees are good at **equality** (`=`), **range** (`<`, `>`, `BETWEEN`),
  **prefix** (`LIKE 'abc%'`), and **sorted retrieval** (`ORDER BY`) — because
  the leaves are stored in sorted order and linked, a range scan is a
  sequential walk across leaf pages after one initial tree descent.
- Each level of the tree corresponds to roughly one page read from disk (or
  buffer pool/cache). Index pages are usually kept hot in memory (InnoDB
  buffer pool / Postgres shared_buffers + OS page cache), so in practice most
  of a B-tree traversal is cache hits, not disk seeks — this is part of why
  indexed lookups are so much faster than a scan.

### Clustered vs non-clustered (heap) storage

This is the single most important conceptual difference between MySQL/InnoDB
and Postgres, and it changes how you think about indexes on each engine.

- **InnoDB (MySQL): clustered by primary key.** The table *is* the primary
  key's B+tree — leaf nodes store the full row, not just a pointer. Every
  other (secondary) index's leaf stores the primary key value, not a disk
  address. So a secondary index lookup is two steps: traverse the secondary
  index to get the PK, then traverse the clustered (PK) index to get the row
  — this is called a **bookmark lookup** or "going through the primary key."
  This is why a good, short, monotonically increasing primary key (e.g. an
  auto-increment `id`, not a random UUID) matters a lot in InnoDB: a random
  PK causes constant page splits and poor locality as rows are inserted
  out of order.
- **Postgres: heap table, all indexes are secondary/non-clustered.** Rows
  live in an unordered heap file; every index (including the one backing the
  primary key) stores a pointer called a **CTID** (page number + offset
  within the page) to the row's physical location. There's no automatic
  "clustering" — `CLUSTER table USING index` can physically reorder the heap
  to match an index once, but it doesn't stay maintained as the table
  changes (subsequent inserts/updates just go back to unordered heap
  placement).
- Practical consequence: in InnoDB, primary key choice affects the physical
  layout and performance of the *whole table*; in Postgres it's "just
  another index" from a storage-layout perspective (though still important
  for lookup speed).

### Index scan vs seek, and how the planner decides

- **Index seek (a.k.a. index range scan):** the engine uses the B-tree to
  jump directly to the matching range and reads only those rows/leaf
  entries — this is the fast path an index exists for.
- **Full table scan (sequential scan):** reads every row. Sometimes actually
  *faster* than using an index — if a query matches a large fraction of the
  table, sequential disk reads beat many scattered random-access index +
  bookmark lookups. The query planner/optimizer decides based on
  **statistics** (row count estimates, value distribution histograms) it
  keeps about each table/index — this is why stale statistics
  (`ANALYZE` in Postgres, `ANALYZE TABLE` in MySQL) can cause the planner to
  pick a bad plan even when the "right" index exists.
- **`EXPLAIN` / `EXPLAIN ANALYZE`** is the tool for seeing what the planner
  actually chose and why — this is the primary way to validate that an index
  is helping (or being used at all) rather than guessing.

---

## How to use them

### Creating and choosing what to index

```sql
CREATE INDEX idx_orders_user_id ON orders (user_id);
CREATE UNIQUE INDEX idx_users_email ON users (email);
```

- Index columns used in `WHERE` filters, `JOIN` conditions, and `ORDER BY` /
  `GROUP BY` — these are the operations an index can directly accelerate.
- **Foreign key columns** should almost always be indexed — the DB usually
  doesn't do this automatically (MySQL/InnoDB does, Postgres does **not**
  auto-index FK columns), and without it, both the join and the FK's
  own referential-integrity check (row lookups on delete/update of the
  parent) do full scans.
- **Selectivity/cardinality** matters: an index on a column with only a
  handful of distinct values relative to row count (e.g. a boolean `status`
  flag on a huge table) is often not useful — the planner may reasonably
  choose a sequential scan over it anyway, because a large fraction of rows
  match any given value.

### Composite (multi-column) indexes

- `CREATE INDEX idx ON t (a, b, c)` builds one B+tree keyed on the
  concatenation `(a, b, c)` in that order — not three separate indexes.
- **Leftmost-prefix rule:** this index can serve queries filtering on `a`
  alone, `a, b`, or `a, b, c`, but **not** `b` alone or `b, c` alone — the
  tree is only sorted by `a` first, so without a value for `a` the engine
  can't narrow the search. Column order in the index definition should match
  your actual query filter order, most-selective/most-commonly-filtered
  columns generally going first (though the "put the equality columns before
  range columns" rule usually matters more than pure selectivity).
- A composite index on `(a, b)` also serves `ORDER BY a, b` for free — the
  leaves are already in that sort order, avoiding a separate sort step.

### Covering indexes and index-only scans

- A **covering index** includes every column a query needs (in the key
  itself, or via `INCLUDE`d columns in Postgres) so the engine never has to
  visit the base table/heap at all — this is called an **index-only scan**
  (Postgres) or a query that's "using index" (MySQL `EXPLAIN` shows `Extra:
  Using index`).
- Postgres: `CREATE INDEX idx ON t (a) INCLUDE (b, c)` — `b, c` are stored in
  the leaf for retrieval but not used for searching/sorting, keeping the
  searchable key smaller while still avoiding a heap fetch.
- Caveat in Postgres: index-only scans additionally require the relevant
  heap pages to be marked all-visible in the **visibility map** (i.e. no
  uncommitted/recently-dead row versions in that page) — otherwise it still
  has to check heap visibility per row (MVCC — see below), which is a common
  surprise when a covering index "should" avoid the heap but the query plan
  still shows heap fetches.

### Partial / filtered indexes

- Index only a subset of rows matching a predicate:
  `CREATE INDEX idx ON orders (id) WHERE status = 'pending';` (Postgres;
  MySQL doesn't support this until functional workarounds/generated
  columns). Much smaller and cheaper to maintain than a full index when
  queries only ever care about a narrow, well-known slice of the table
  (e.g. unprocessed jobs in a queue table where most rows are terminal).

### Expression / functional indexes

- Index the result of an expression rather than a raw column:
  `CREATE INDEX idx ON users (LOWER(email));` — lets a query like
  `WHERE LOWER(email) = 'x@y.com'` use the index, whereas an index on the
  raw `email` column can't match a query wrapping the column in a function
  (applying a function to an indexed column generally makes the column
  unindexable for that predicate — this is called a **non-sargable**
  predicate).

### Unique indexes and constraints

- A `UNIQUE` index both enforces the constraint and speeds up lookups — it's
  the mechanism behind `PRIMARY KEY` and `UNIQUE` constraints under the
  hood, not a separate feature.

---

## Trade-offs

### Reads vs writes

- Every `INSERT`/`UPDATE`/`DELETE` has to update every index on the affected
  columns, not just the table — so more indexes means slower writes and more
  WAL/redo-log volume. A table with 10 indexes pays that cost on every
  write regardless of which columns actually changed (an `UPDATE` that
  touches an indexed column always rewrites that index entry; in Postgres,
  updating *any* column can still force new index entries for all indexes
  because MVCC writes a new row version — mitigated somewhat by **HOT
  updates**, which skip index updates when the new row version fits in the
  same heap page and no indexed column changed).
- The right number of indexes is workload-dependent: an OLTP table with
  heavy writes and few known query shapes wants few, carefully chosen
  indexes; a reporting/OLAP-style table that's write-light and query-heavy
  can afford many.

### Storage cost

- Indexes take disk space, sometimes comparable to or larger than the table
  itself (especially wide composite or covering indexes) — this also means
  more data to keep hot in the buffer pool/cache, competing with the table's
  own pages.

### Index bloat and maintenance

- **Postgres:** MVCC means updates/deletes leave dead tuples behind (old row
  versions); indexes accumulate dead entries too until `VACUUM` reclaims
  them. A heavily updated table with infrequent vacuuming gets bloated
  indexes that are larger and slower than they should be — `autovacuum`
  tuning is a real, common operational concern.
- **MySQL/InnoDB:** doesn't have Postgres-style heap bloat (updates rewrite
  in place more often), but secondary indexes still fragment over time and
  benefit from periodic `OPTIMIZE TABLE` (which rebuilds the table/indexes).
- Both engines need statistics refreshed (`ANALYZE`) after significant data
  changes, or the planner's cardinality estimates go stale and it picks
  worse plans — a frequently-missed cause of "the index exists but isn't
  being used."

### Over-indexing vs under-indexing

- **Under-indexed:** slow reads, sequential scans on large tables, DB CPU
  and I/O dominated by table scans.
- **Over-indexed:** slow writes, bloated storage, and — less obviously — the
  planner sometimes has *more* plans to choose from and occasionally picks a
  worse one, plus more indexes competing for buffer pool space can push hot
  data out of cache.
- The practical process: index based on actual slow-query logs / `EXPLAIN
  ANALYZE` evidence, not speculation; periodically look for unused indexes
  (`pg_stat_user_indexes` in Postgres, `sys.schema_unused_indexes` in MySQL)
  and drop them — an index that's never used is pure write/storage cost with
  no benefit.

---

## Advanced concepts

### Index types beyond B-tree

- **Hash indexes:** O(1) equality lookup, but no range scan or sort support
  at all (a hash has no ordering). Postgres has an explicit `USING hash`
  index type (WAL-logged and crash-safe since PG10); MySQL's `MEMORY` engine
  uses hash indexes by default, InnoDB doesn't expose them directly but
  maintains an internal **adaptive hash index** — a self-tuning, automatic
  in-memory hash layer built on top of frequently-accessed B-tree pages to
  speed up equality lookups, transparent to the user.
- **GiST / SP-GiST (Postgres):** generalized search trees for
  non-scalar/geometric data types — used for full-text search ranking,
  geometric types, range types, and as the base for extensions like
  `pg_trgm` (trigram similarity search) and PostGIS.
- **GIN (Generalized Inverted Index, Postgres):** built for indexing values
  that contain multiple component values — array elements, JSONB keys,
  full-text search lexemes. Instead of one entry per row, it indexes each
  component and maps it back to the row(s) containing it — essentially an
  inverted index, like a search engine's. This is what makes
  `jsonb_column @> '{"key": "value"}'` or full-text `tsvector @@ tsquery`
  queries fast.
- **BRIN (Block Range Index, Postgres):** stores min/max summaries per block
  range instead of per-row entries — tiny index size, works well on very
  large tables where the indexed column correlates with physical insertion
  order (e.g. a timestamp column on an append-only log table), but
  degrades badly if the data isn't physically correlated with the index key.
- **Spatial indexes (R-tree-based):** MySQL's spatial indexes on `GEOMETRY`
  columns, and Postgres GiST-based indexes via PostGIS, for bounding-box and
  nearest-neighbor queries on geographic/geometric data.

### MVCC interaction (Postgres specifically)

- Because Postgres implements MVCC by keeping multiple row versions rather
  than in-place updates plus an undo log (InnoDB's approach), index entries
  can point to row versions that are no longer visible to the current
  transaction — hence the visibility-map check for index-only scans
  mentioned above, and why long-running transactions (which prevent vacuum
  from cleaning up old versions) are a common cause of index/table bloat.

### Locking during index builds

- Building an index traditionally locks the table against writes for the
  duration. Both engines have online/concurrent variants for production use:
  Postgres's `CREATE INDEX CONCURRENTLY` (slower, takes two table scans, but
  doesn't block writes) and MySQL's `ALGORITHM=INPLACE` (or `INSTANT` for
  some operations) with InnoDB online DDL. Forgetting to use these on a
  large, live production table is a classic outage cause.

### Index-merge / bitmap scans

- When no single index covers a multi-condition query well, both engines can
  combine multiple single-column indexes: MySQL's **index merge**
  (`union`/`intersection` of row-ID sets from separate index scans) and
  Postgres's **bitmap index scan** (build an in-memory bitmap of matching
  heap pages from one or more indexes, `AND`/`OR` the bitmaps together, then
  do one sorted pass over the heap) — this is why you don't always need a
  perfect composite index for every combination of filters, though a
  purpose-built composite index is still generally faster.

### Descending and mixed-order indexes

- `CREATE INDEX idx ON t (a ASC, b DESC)` — matters when a query sorts
  columns in different directions (e.g. `ORDER BY created_at DESC, id ASC`);
  without a matching mixed-order index the engine needs an explicit sort
  step even though an index on the columns exists.

### Invisible / disabled indexes (testing removal safely)

- MySQL 8+ supports `ALTER TABLE t ALTER INDEX idx INVISIBLE` — hides an
  index from the planner without dropping it, so you can verify nothing
  regresses before actually deleting it (and can instantly reverse it if
  something does).

---

## Other important things to know

- **Index maintenance is not free even when "just reading":** a covering
  index avoiding heap access still needs the index itself kept current on
  every write — there's no such thing as a free covering index, only a
  favorable tradeoff for read-heavy tables.
- **Don't index columns with function calls or implicit type casts** in the
  predicate unless there's a matching expression index — this silently
  defeats an otherwise-correct index (non-sargable queries).
- **Foreign key + cascading delete performance:** an unindexed FK column on
  the *child* table makes every parent-row delete/update with
  `ON DELETE CASCADE` do a full scan of the child table to find rows to
  cascade — a very common, very avoidable production slowdown.
- **Index size vs cache:** the working set of *frequently used* indexes
  needs to fit comfortably in memory (buffer pool/shared_buffers) for
  indexes to deliver their expected latency — an index that's technically
  present but constantly evicted from cache degrades toward disk-seek
  latency.

---

## Relational vs non-relational (NoSQL) indexes: what's actually different

The underlying data structure ideas (B-trees, hashing, inverted indexes) are
shared across both worlds — the real differences are about **what the
database lets you index, how indexes relate to physical data layout, and
what consistency guarantee you get when reading through one.**

| | Relational (MySQL/Postgres) | Non-relational (e.g. DynamoDB, MongoDB, Cassandra) |
|---|---|---|
| Primary key = physical layout? | InnoDB: yes (clustered). Postgres: no (heap + pointers). | Usually yes and rigid — the partition key *is* the sharding/placement mechanism, not just a lookup aid. |
| Ad-hoc secondary indexes | Freely add any index on any column combination, anytime (mostly). | Often constrained: DynamoDB GSIs/LSIs must be explicitly designed in, with limits (max count, creation-time-only for LSIs); MongoDB is closer to relational flexibility here. |
| Joins across indexes | Query planner can combine multiple indexes (bitmap/index-merge) and join across tables. | Typically no server-side joins; secondary indexes usually exist to avoid needing one, by denormalizing instead. |
| Consistency of index reads | Indexes are transactionally consistent with the table (same MVCC/lock semantics) in both engines. | Often **eventually consistent** — e.g. DynamoDB GSIs are always eventually consistent with the base table, by design, because they're asynchronously replicated to potentially different partitions/nodes. |
| Query planner | Cost-based optimizer chooses among available indexes automatically. | Often no optimizer at all — you must explicitly query via a specific index/table; "wrong" access patterns are often simply inexpressible rather than just slow. |
| Design-time vs runtime | Index what you need as query patterns emerge; schema stays normalized. | Access patterns are typically designed in upfront (see single-table design in DynamoDB) — indexes/denormalization *are* the schema design, not an afterthought. |

### The key conceptual difference

In a relational DB, indexes are an **optional performance accelerant** layered
on top of a normalized schema — you can add/drop them without changing what
queries are *possible* (just their speed), and the planner picks among them
automatically. In many NoSQL systems (especially key-value/wide-column ones
like DynamoDB or Cassandra), indexes and key design **are** the schema: what
you can query is defined by which indexes exist, there's no query planner to
fall back to a scan-and-hope, and getting the key/index design wrong at
day one often means an expensive migration later rather than just "add an
index." This is the same idea covered in more depth in
[dynamodb.md](dynamodb.md) under single-table design — it's really an
index-design discipline forced on you by the absence of joins and ad-hoc
query planning, not a NoSQL-specific quirk.

Document stores like MongoDB sit in between: they support ad-hoc secondary
indexes (including compound, partial, and multi-key indexes on array
fields — structurally similar to a relational composite or GIN index) fairly
freely, closer to relational ergonomics, but still lack a cross-collection
join-optimizing planner the way a relational engine does (MongoDB's
`$lookup` exists but isn't optimized the way a relational join is).

Search-oriented and columnar/OLAP systems add index types with no real
relational-OLTP equivalent at all — e.g. Elasticsearch's inverted indexes
built for full-text relevance ranking, or columnar stores (Redshift,
BigQuery, ClickHouse) that skip traditional row indexes entirely in favor of
column-wise storage plus zone maps/min-max block pruning (conceptually close
to Postgres's BRIN index, but as the *default* storage strategy rather than
an optional index type).
