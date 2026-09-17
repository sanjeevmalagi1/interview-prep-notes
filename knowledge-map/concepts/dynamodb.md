# DynamoDB

## Overview

DynamoDB is AWS's fully managed, serverless key-value / wide-column NoSQL
database. It's built on the ideas from Amazon's original 2007 Dynamo paper
(consistent hashing, vector clocks, gossip protocol, quorum reads/writes) but
re-implemented as a managed service with a simpler consistency model (Dynamo
used last-write-wins-ish multi-version reconciliation; DynamoDB gives you a
single authoritative value per key via Multi-Paxos-based replication).

It trades the flexibility of relational databases (joins, ad-hoc queries,
secondary access patterns for free) for near-infinite horizontal scalability
and predictable, single-digit-millisecond latency at any scale — but only if
you design your table around known access patterns up front.

**Rule of thumb:** reach for DynamoDB when access patterns are known in
advance, you need predictable low-latency reads/writes at massive scale, and
you're willing to denormalize. Reach for a relational database when you need
ad-hoc queries, complex joins, or strong multi-row/multi-table transactions
as the common case.

---

## How data is stored

### Tables, items, attributes

- A **table** is a collection of **items** (≈ rows). Each item is a
  collection of **attributes** (≈ columns), but unlike a relational row,
  items in the same table don't need the same attributes — DynamoDB is
  schemaless except for the key attributes.
- Attributes have types: scalars (String, Number, Binary, Boolean, Null),
  sets (String Set, Number Set, Binary Set), and documents (List, Map) —
  so an item can nest JSON-like structures.
- Item size limit: **400 KB** (includes attribute names + values).

### Partitions

- Under the hood, a table's data is horizontally split across **partitions**
  — each partition is a chunk of SSD-backed storage (roughly 10 GB) with its
  own throughput allocation.
- DynamoDB decides which partition an item lives on by hashing the item's
  **partition key** value: `partition = hash(partition_key) mod N`. This is
  **consistent hashing** conceptually — items with the same partition key
  always land on the same partition, and hashing spreads keys evenly across
  partitions to avoid hot spots (assuming the key has good cardinality).
- Within a partition, data is stored **sorted by sort key** (if the table has
  one) as a B-tree-like structure, so range queries on the sort key within a
  partition key are efficient.
- Partitions are added automatically as data grows or as throughput demands
  increase (partition splits) — this is why DynamoDB doesn't require
  capacity planning the way a self-managed sharded DB would.
- Each item is replicated **synchronously across 3 Availability Zones**
  within a region (using a Multi-Paxos-based protocol for leader election and
  write replication) — this is the basis for its durability and availability
  guarantees.

---

## What makes it fast

1. **O(1) partition routing.** The partition key is hashed directly to a
   physical partition — no query planner, no index scan, no joins. A
   `GetItem` or `Query` by partition key goes straight to the right node.

2. **No cross-partition coordination for normal reads/writes.** Because each
   item lives entirely on one partition (replicated across 3 AZs, not
   sharded further), a read or write only ever talks to the partition(s)
   owning that key — latency doesn't grow with table size.

3. **SSD-backed storage** for all table data, with sort keys stored in
   sorted order per partition, so range scans within a partition key are
   sequential disk reads, not random seeks.

4. **Provisioned/adaptive throughput isolates noisy neighbors.** Each
   partition gets its own slice of the table's throughput; adaptive
   capacity and (with on-demand mode) automatic partition splitting prevent
   one hot key from starving others.

5. **No joins, no foreign keys, no multi-table queries.** The query engine
   is intentionally dumb — it can only do the things the key schema and
   indexes allow. This constraint is *why* it's fast: there's no
   possibility of an expensive query plan, because expensive query shapes
   simply aren't expressible. This is the central tradeoff of DynamoDB (and
   NoSQL wide-column stores generally): you pay the modeling cost at design
   time instead of the query cost at run time.

6. **DAX (DynamoDB Accelerator)** — an optional in-memory write-through
   cache layer in front of DynamoDB for microsecond read latency on hot
   items, without changing application code (same API).

---

## Keys: partition key, sort key, primary key

- **Partition key (a.k.a. hash key):** determines which partition the item
  lives on. Required on every table. Should have high cardinality and even
  access distribution to avoid "hot partitions."
- **Sort key (a.k.a. range key):** optional. If present, the **primary key**
  is the composite `(partition key, sort key)` — multiple items can share
  the same partition key as long as their sort keys differ. Items with the
  same partition key are stored together, sorted by sort key.
  - Without a sort key: primary key = partition key alone, one item per key
    value (simple key).
  - With a sort key: primary key = partition key + sort key (composite key),
    enabling one-to-many item collections under a single partition key —
    this is the main lever used in single-table design.
- **Query vs Scan:**
  - `GetItem` — fetch one item by exact primary key.
  - `Query` — fetch items sharing a partition key, optionally filtered by a
    sort key condition (`=`, `<`, `>`, `BETWEEN`, `begins_with`). Efficient —
    touches only the relevant partition.
  - `Scan` — reads every item in the table (or index), filtering after the
    read. Expensive, avoid in hot paths; useful for exports/analytics.
- **Sort key tricks:** because `begins_with` and range conditions work
  lexically, sort keys are often composite strings like
  `"ORDER#2024-01-15#<orderId>"` or `"v1#profile"` to encode multiple
  hierarchical or type dimensions into one sortable field.

---

## Secondary indexes

Base table access is limited to lookups by the primary key. Secondary
indexes let you query by other attributes — at the cost of extra storage and
(for GSIs) eventual consistency and separate throughput.

### Local Secondary Index (LSI)

- Same partition key as the base table, **different sort key**.
- Must be created **at table creation time** — cannot be added later.
- Shares the base table's partition (and its 10 GB-per-partition-key
  soft-ish limit — actually a hard limit: all items + LSI projections for one
  partition key must fit in 10 GB).
- Reads can be strongly or eventually consistent (your choice), same as the
  base table, because it lives on the same physical partition.
- Max 5 LSIs per table.

### Global Secondary Index (GSI)

- **Different partition key** (and optionally different sort key) from the
  base table — effectively a second table, replicated asynchronously from
  the base table.
- Can be added or removed at any time after table creation.
- Has **its own provisioned throughput** (or scales independently in
  on-demand mode) — a GSI with insufficient throughput can throttle writes
  to the base table.
- Always **eventually consistent** — there's a small replication lag between
  a base table write and its visibility in the GSI.
- Max 20 GSIs per table (soft limit, raisable).
- This is the main tool for supporting multiple access patterns on one
  table — e.g., base table keyed by `userId`, GSI keyed by `email` to
  support login lookups.

### Projections

Both index types let you choose which attributes get copied into the index:
`KEYS_ONLY`, `INCLUDE` (specific attributes), or `ALL`. Smaller projections
= less storage and lower write cost, but may require a follow-up `GetItem`
to the base table if you need attributes not in the projection.

---

## Capacity modes

- **Provisioned throughput:** you specify Read Capacity Units (RCU) and
  Write Capacity Units (WCU) up front. 1 RCU = one strongly consistent read
  of up to 4 KB/sec (or two eventually consistent reads); 1 WCU = one write
  of up to 1 KB/sec. Supports auto-scaling policies. Cheaper for predictable,
  steady workloads.
- **On-demand:** pay per request, no capacity planning; DynamoDB scales
  instantly (within reason — very steep spikes can still throttle briefly
  right after a big jump). Better for unpredictable/spiky traffic.

## Consistency model

- **Eventually consistent reads** (default): may read slightly stale data
  (replication lag across the 3 AZ replicas), but cheapest and highest
  throughput.
- **Strongly consistent reads:** always reflect the latest successful write
  (reads from/through the leader replica); cost 2x the RCUs, unavailable on
  GSIs, and slightly higher latency.
- **Transactions** (`TransactWriteItems`/`TransactGetItems`): ACID
  transactions across up to 100 items/25 in older limits spanning multiple
  tables, using two-phase commit. Costs 2x the normal capacity (one set for
  prepare, one for commit).

---

## Advanced: single-table design

The signature DynamoDB modeling pattern, and the thing that most confuses
people coming from relational databases.

### The idea

Instead of one DynamoDB table per entity type (like SQL tables), you put
**every entity type in one table**, and use generic, overloaded key names
(commonly `PK` / `SK`) whose *meaning* changes depending on the item type.
Related entities are colocated under the same partition key so that a single
`Query` (one round trip) retrieves an entire object graph — e.g., an order
and all its line items, or a user and all their recent sessions.

Example (e-commerce):

| PK | SK | attributes |
|---|---|---|
| `USER#123` | `PROFILE` | name, email, ... |
| `USER#123` | `ORDER#2024-01-15#A1` | total, status, ... |
| `USER#123` | `ORDER#2024-01-20#B2` | total, status, ... |
| `ORDER#A1` | `ITEM#sku001` | qty, price |
| `ORDER#A1` | `ITEM#sku002` | qty, price |

A `Query` for `PK = USER#123` returns the profile and all orders in one
request, sorted by `SK`. A `Query` for `PK = USER#123 AND SK begins_with
ORDER#2024-01` returns just January's orders — the composite sort key
encodes a range filter.

### Why: it's a consequence of no joins

Relational design normalizes data and relies on joins at query time to
reassemble related rows. DynamoDB has no joins and charges per-partition
I/O, so single-table design **denormalizes at write time** — you decide the
access patterns first, then shape the keys so that every important query
becomes a single `Query` against one partition (or a `GetItem`), never a
`Scan` or an application-level fan-out across multiple tables.

### Techniques that go with it

- **Overloaded GSIs (inverted index pattern):** a GSI with `SK` as its
  partition key and `PK` as its sort key lets you query "the other
  direction" — e.g., find all users who ordered a given SKU.
- **Item type prefixes:** prefixing keys (`USER#`, `ORDER#`, `ITEM#`) lets
  heterogeneous entity types share the same physical key attributes without
  collisions, and lets a `Query` with `begins_with` fetch just one entity
  type from a partition.
- **Sparse indexes:** a GSI only indexes items that actually have the GSI's
  key attributes — so you can create a GSI that only contains, say, "orders
  with status = PENDING" by only setting a `GSI1PK` attribute on those
  items. Cheap way to model a filtered/materialized view.
- **Write sharding:** for very hot partition keys (e.g., a global counter or
  a celebrity user), append a random or hashed suffix (`SHARD#0`..`SHARD#9`)
  to spread writes across multiple physical partitions, then fan-out reads
  and aggregate client-side.
- **Adjacency list pattern:** modeling many-to-many relationships (like a
  graph) by storing both "forward" and "reverse" edges as items, similar to
  an adjacency list in graph theory.

### The tradeoff

Single-table design optimizes for a known, finite set of access patterns at
the cost of:
- Much harder to reason about / self-document (the schema is implicit in
  the application code, not the table structure).
- Poor fit for ad-hoc analytics or reporting — usually solved by streaming
  changes out via **DynamoDB Streams** into a separate OLAP store (Redshift,
  Athena over S3, OpenSearch) rather than querying DynamoDB directly.
- New/changed access patterns discovered after launch can require
  backfills or new GSIs, not just a new SQL query.

**When to use multi-table instead:** when access patterns are genuinely
unpredictable, when different entities have very different scaling/traffic
profiles (so you want independent throughput and monitoring), or when the
team's velocity benefits more from a simpler, self-documenting schema than
from minimizing round trips. Many production systems use a hybrid — a few
single-table "hot path" tables plus separate tables for less coupled data.

---

## Other important concepts

- **DynamoDB Streams:** an ordered, time-limited (24h) change log of item-
  level modifications (insert/update/delete, with before/after images
  optionally). Powers event-driven architectures (Lambda triggers),
  cross-region replication (Global Tables), and CQRS-style read models.
- **Global Tables:** multi-region, multi-active replication built on
  Streams — last-writer-wins conflict resolution based on timestamps. Gives
  active-active geo-replication without the app managing it.
- **TTL (Time to Live):** mark items to auto-expire; DynamoDB deletes them
  in the background (within ~48h of expiry) at no extra write cost — useful
  for session data, caches, or event logs living alongside permanent data
  in a single-table design.
- **Conditional writes & optimistic locking:** `ConditionExpression` on
  `PutItem`/`UpdateItem` (e.g., `attribute_not_exists(PK)` or a version
  number check) gives compare-and-swap semantics without needing
  distributed locks — the standard way to implement optimistic concurrency
  control.
- **Hot partition / hot key problem:** the most common DynamoDB production
  incident. Throughput is provisioned per-partition, so a skewed key
  distribution (one tenant, one popular item) can throttle even when the
  table's *aggregate* capacity looks fine. Adaptive capacity helps
  automatically, but write sharding is the manual fix for extreme cases.
- **Cost model:** pay for storage + RCU/WCU (or on-demand request units) +
  Streams reads + backup/PITR — no cost for idle compute, which is part of
  why it's popular for serverless (Lambda-backed) architectures.
