# In-Memory Data Stores

## Overview

In-memory data stores keep the primary copy of data in RAM instead of on disk,
trading durability guarantees for very low latency (sub-millisecond) and very
high throughput. They're used for caching, session storage, rate limiting,
leaderboards, pub/sub messaging, and sometimes as a primary datastore for
ephemeral or recomputable data.

| | Redis | Memcached |
|---|---|---|
| Data model | Rich (strings, hashes, lists, sets, sorted sets, streams, bitmaps, HyperLogLog, geo) | Simple key-value (strings/blobs only) |
| Persistence | Optional (RDB snapshots, AOF log) | None — pure cache, data lost on restart |
| Threading | Single-threaded command execution (I/O threading added in 6+) | Multi-threaded from the start |
| Replication / HA | Built-in (replicas, Sentinel, Cluster) | None built-in (client-side sharding only) |
| Clustering | Redis Cluster (hash slots) | Client-side consistent hashing |
| Transactions | MULTI/EXEC, Lua scripting | None |
| Pub/Sub, streams | Yes | No |
| Max value size | 512MB | 1MB (default) |
| Typical use | Cache + broker + data structure server | Pure, dumb, fast cache |

**Rule of thumb:** reach for Memcached when you want the simplest possible
distributed cache and nothing else. Reach for Redis when you need richer data
structures, persistence, pub/sub, or you want one system to do more than
caching (queues, locks, counters, leaderboards).

---

## Redis

### Architecture

- Single-threaded **event loop** (epoll/kqueue) handles all client commands —
  this is what makes commands atomic without explicit locking: no two
  commands can interleave mid-execution.
- Since Redis 6, **I/O threading** was added: reading/parsing client input and
  writing output can happen on multiple threads, but command *execution*
  itself still happens on the single main thread. This relieves network I/O
  as a bottleneck without giving up the atomicity guarantee.
- Redis 7's multi-threading extends to background tasks (expiration, RDB/AOF
  writing) via `fork()`-based child processes, not the command path.

### Persistence

- **RDB (snapshotting):** point-in-time binary dump on a schedule or on
  demand (`SAVE`/`BGSAVE`). Compact, fast to restore, but can lose data
  between snapshots.
- **AOF (append-only file):** logs every write command; replayed on restart.
  More durable (configurable fsync: always / every second / never), larger
  file, slower to restart with.
- Often run together: AOF for durability, RDB for fast full backups/restore.

### Concurrency model

- Because command execution is single-threaded, **every individual command
  is atomic** — no interleaving, no need for application-level locks for a
  single command.
- Multi-key or multi-step operations still need explicit atomicity — see
  Transactions below.
- Under the hood, `fork()` is used for RDB snapshotting and AOF rewriting so
  the main thread isn't blocked; relies on copy-on-write memory pages.

### Transactions

- `MULTI` / `EXEC` queues a batch of commands and executes them atomically
  and sequentially — no other client's commands can interleave. It is **not**
  a rollback-capable transaction: if a command fails at runtime (e.g. wrong
  type), the rest still execute; there's no "abort and undo."
- `WATCH` provides **optimistic locking**: watch one or more keys, and if any
  watched key changes before `EXEC`, the whole transaction is aborted
  (`EXEC` returns nil) — classic compare-and-swap pattern.
- **Lua scripting** (`EVAL`/`EVALSHA`, and `FUNCTION` in Redis 7+) is the more
  powerful alternative: an entire script runs atomically as if it were one
  command, with real conditional logic — this is how most "real" transactions
  are implemented in practice (e.g. distributed locks, rate limiters).

### Other notable concepts

- **Eviction policies** (when `maxmemory` is hit): `noeviction`, `allkeys-lru`,
  `volatile-lru`, `allkeys-lfu`, `volatile-ttl`, `allkeys-random`, etc.
- **Replication:** async by default (leader → replicas); `WAIT` can force
  synchronous acknowledgment from N replicas.
- **Sentinel:** monitors master/replica sets and handles automatic failover
  for non-clustered deployments.
- **Redis Cluster:** shards data across nodes using 16384 hash slots;
  supports resharding and partial availability during partitions.
- **Pub/Sub & Streams:** `PUBLISH`/`SUBSCRIBE` for fire-and-forget messaging;
  `XADD`/`XREAD`/consumer groups for durable, replayable message streams
  (closer to Kafka-lite semantics).
- **Distributed locks (Redlock):** algorithm for acquiring a lock across
  multiple independent Redis nodes to tolerate single-node failure —
  controversial (Martin Kleppmann's critique) around its safety guarantees
  under clock drift/GC pauses.
- **Keyspace notifications:** pub/sub events on key expiry/changes, useful
  for building reactive systems (e.g. "notify on TTL expiry").
- **Lazy freeing (`UNLINK`):** deletes large keys asynchronously in a
  background thread so the main thread isn't blocked reclaiming memory.

---

## Memcached

### Architecture

- **Multi-threaded** from day one (`libevent` + a thread pool) — designed
  purely to maximize throughput for a simple key-value workload across CPU
  cores.
- Uses a **slab allocator**: memory is divided into fixed-size chunks
  (slabs) grouped by size classes, to avoid fragmentation from
  variable-length values. Downside: can waste memory if value sizes don't
  fit slab classes well ("slab calcification").

### Concurrency model

- Multiple worker threads handle client connections in parallel, each with
  its own event loop; a global (or per-slab-class, in modern versions) lock
  protects the shared hash table and LRU structures.
- Because it has no rich operations or scripting, concurrency correctness is
  simpler: operations are single-key, and atomicity is limited to primitives
  like `incr`/`decr`/`cas` (compare-and-swap using a version token) —
  there's no concept of multi-command transactions at all.

### No persistence, no replication

- Purely volatile: a restart or eviction means the data is gone. This is by
  design — Memcached assumes the source of truth lives elsewhere (a
  database) and it is only ever a cache in front of it.
- No built-in replication or clustering — high availability and sharding are
  pushed to the client (consistent hashing across a list of nodes) or to a
  proxy layer (e.g. `mcrouter`).

### Other notable concepts

- **CAS (check-and-set):** `gets` returns a value + a CAS token; `cas` writes
  only succeed if the token still matches — Memcached's only concurrency
  primitive beyond atomic increment/decrement.
- **Consistent hashing:** since there's no server-side clustering, clients
  (or a proxy like `mcrouter`/`twemproxy`) hash keys to nodes; consistent
  hashing minimizes cache misses when nodes are added/removed.
- **Item size limit:** default 1MB per value (configurable), enforced to
  keep the slab allocator efficient.

---

## Cross-cutting advanced topics

### Cache invalidation strategies
- **TTL-based expiry** — simplest, but can serve stale data.
- **Write-through / write-behind / cache-aside (lazy loading)** — patterns
  for keeping cache and source-of-truth database in sync.
- **Thundering herd / cache stampede:** many clients simultaneously miss on
  a hot key's expiry and all hit the DB at once. Mitigations: request
  coalescing, probabilistic early expiration, locking/single-flight on
  recompute.

### Consistency
- These stores generally offer **no cross-node strong consistency** by
  default. Redis replication is asynchronous (risk of losing recent writes
  on failover); Redis Cluster and Memcached sharding are effectively
  eventually-consistent with respect to node failures.
- Contrast with systems like ZooKeeper/etcd that trade throughput for
  linearizable consistency — in-memory caches deliberately don't do this.

### Memory management differences
- Redis uses a more general allocator (`jemalloc` by default) with
  per-key/value overhead tracked individually — flexible but more prone to
  fragmentation on very heterogeneous key/value sizes.
- Memcached's slab allocator trades some memory efficiency for predictable,
  fragmentation-resistant allocation performance.

### Newer/adjacent tools worth knowing about
- **Dragonfly / KeyDB:** modern Redis-protocol-compatible stores that are
  natively multi-threaded (unlike core Redis), aiming for much higher
  throughput per node on multi-core machines.
- **Valkey:** the Linux Foundation-backed fork of Redis (post-license-change
  in 2024), drop-in compatible, now what most cloud providers ship as their
  "Redis" managed offering.
- **Hazelcast / Apache Ignite:** in-memory data grids — go further than
  caching, offering distributed compute, near-caches, and strong
  partitioning models.
