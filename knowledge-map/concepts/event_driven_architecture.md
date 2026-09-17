# Event-Driven Architecture

## Overview

Event-Driven Architecture (EDA) is a design style where services communicate
by producing and consuming **events** — immutable facts about something that
already happened (`OrderPlaced`, `PaymentCaptured`, `InventoryReserved`) —
instead of calling each other directly. Producers don't know who (if anyone)
consumes their events, and consumers react asynchronously whenever an event
arrives. This inverts the dependency direction of a typical request/response
call: in EDA the producer depends on nothing downstream, and any number of
consumers can be added later without the producer changing at all.

**Rule of thumb:** reach for EDA when you have multiple, independently
deployed consumers that need to react to the same fact (fan-out), or when
producer and consumer availability/throughput must be decoupled (the
consumer can be down or slow without blocking the producer). It's overkill
for a simple synchronous need-the-answer-now call between two services —
that's just a regular RPC/HTTP call in disguise if you force it through an
event.

---

## Core concepts

### Event

An immutable record that something happened, named in the past tense
(`OrderPlaced`, not `PlaceOrder` — that's a command). An event carries enough
data for consumers to act, or just an ID/pointer for consumers to fetch more
(see "thin vs. fat events" below).

### Event vs. command vs. message

- **Command** — an instruction to do something (`ChargeCard`), sent to one
  specific handler that is expected to execute it. Imperative, targeted.
- **Event** — a fact that already happened, broadcast to whoever cares.
  Nobody is "told" to do anything; consumers choose to react.
- **Message** — the umbrella term (the envelope on the wire); commands and
  events are both kinds of messages.

### Producer / consumer decoupling

Producers publish without knowing which (or how many) consumers exist.
Consumers subscribe without knowing which service produced the event. This
is what lets you add a new consumer (e.g., a fraud-detection service
listening to `OrderPlaced`) without touching the producer or any existing
consumer — the main structural payoff of EDA.

### Event broker / log

The middleware that durably stores and routes events between producers and
consumers — a message queue (RabbitMQ, SQS) or a distributed log (Kafka,
Pulsar). The broker is what makes producer/consumer decoupling durable:
events aren't lost if a consumer is temporarily down.

### Choreography vs. orchestration

- **Choreography** — no central coordinator; each service reacts to events
  and emits its own events, and the overall workflow emerges from these
  local reactions chained together. Highly decoupled, but the end-to-end
  flow is implicit — hard to see "the order process" in one place, hard to
  debug where a multi-step flow got stuck.
- **Orchestration** — a central coordinator (an orchestrator, or a saga
  manager) explicitly calls each step and tracks the workflow's state.
  Easier to reason about and debug, but reintroduces a central component
  that knows about every participant.

Most real systems mix both: choreography for simple, local reactions;
orchestration (sagas) for multi-step business transactions that need
compensation logic.

### Delivery guarantees

- **At-most-once** — event may be lost, never redelivered. Rarely what you
  want.
- **At-least-once** — event is redelivered until acknowledged; consumers
  must be **idempotent** (safe to process the same event twice) because
  duplicates *will* happen (retry after a crash between processing and ack).
  This is the practical default for almost every broker.
- **Exactly-once** — effectively at-least-once delivery + idempotent
  processing (often via a dedup key or a transactional outbox on the
  consumer side); true end-to-end exactly-once across independent systems
  is not achievable in general, so "exactly-once" in marketing usually means
  "exactly-once effect," not "exactly-once delivery."

### Ordering

Most brokers only guarantee order **within a partition/shard**, not
globally. If ordering matters (e.g., events for the same `orderId` must be
processed in sequence), you must key/partition events by that entity's ID so
all its events land on the same partition and are consumed by the same
consumer instance in order.

### Schema evolution

Events are a long-lived contract between services that may deploy on
different schedules, so consumers must tolerate new optional fields
(backward compatibility) and producers must avoid breaking changes (renaming
or removing fields) without a versioning strategy. A schema registry
(Confluent Schema Registry, AWS Glue Schema Registry) with Avro/Protobuf
enforces this at write time instead of relying on discipline.

### Thin vs. fat events

- **Thin/notification event** — carries just an ID (`{"orderId": "123"}`);
  consumer calls back to the producer's API to fetch full data. Simpler
  payload, but couples consumer to producer's availability and adds a
  network hop.
- **Fat/state-carrying event** — carries the full relevant state in the
  event itself. Consumer never calls back, fully decoupled at runtime, but
  payloads are larger and every field change requires a schema update.

Most systems land on fat events for anything performance-sensitive, thin
events for large blobs that are usually not needed to react to the event.

---

## Implementing EDA with Apache Kafka

Kafka is a distributed, partitioned, replicated commit log — not a
traditional queue. Events published to a **topic** are appended to a log and
retained for a configurable period (or forever, with compaction), and
multiple independent consumers can each read the full log at their own pace
without deleting or affecting one another. This "log as source of truth"
model is what makes Kafka the default backbone for EDA at scale (vs. a
queue like SQS/RabbitMQ, which is more suited to work-distribution/task
queues than to broadcasting durable facts).

### Key building blocks

- **Topic** — a named stream of events (`orders.order-placed`). Split into
  **partitions** for parallelism; each partition is an ordered, append-only
  log.
- **Partition key** — determines which partition an event lands on
  (`hash(key) % numPartitions`). Use the entity ID you need ordering for
  (e.g., `orderId`) as the key so all events for that entity are ordered and
  consumed by one consumer instance.
- **Producer** — appends events to a topic. Configurable durability via
  `acks` (`acks=all` waits for all in-sync replicas — use this when you
  can't afford to lose an event).
- **Consumer group** — a set of consumer instances that split a topic's
  partitions between them for parallel processing; each partition is
  consumed by exactly one instance within a group at a time, but every group
  gets its own full copy of the stream (this is how Kafka supports many
  independent consumers of the same events).
- **Offset** — a per-partition, per-consumer-group pointer to "how far
  we've read." Committing offsets (usually after successful processing, not
  before) is what gives at-least-once delivery — a crash before commit means
  the event is redelivered.
- **Retention / compaction** — events are retained for a time window
  (replay is possible by resetting offsets) or, with **log compaction**,
  Kafka keeps only the latest event per key forever — this turns a topic
  into a durable, replayable snapshot of "current state per key," which is
  exactly what backs the outbox and CDC patterns below.

### Practical implementation notes

- **Idempotent consumers**: store a processed-event ID (or use the
  event's natural key + a dedup table) before acting, since Kafka's default
  delivery is at-least-once and redelivery after a consumer crash is normal,
  not exceptional.
- **Idempotent producers / transactions**: Kafka supports
  `enable.idempotence=true` (dedups retries at the producer level) and
  transactions (atomically write to multiple partitions/topics, useful for
  "consume from topic A, produce to topic B" exactly-once pipelines via
  Kafka Streams).
- **Dead-letter topics**: route events that repeatedly fail processing to a
  separate `*.dlq` topic instead of blocking the partition forever or
  silently dropping them, so they can be inspected/replayed later.
- **Schema registry + Avro/Protobuf**: enforce compatible schema evolution
  at produce time rather than discovering a breaking change in a downstream
  consumer's logs.
- **Consumer lag** is the key operational metric — how far behind a consumer
  group is from the head of the log; alert on it to catch stuck or
  under-scaled consumers.

---

## Patterns

### Transactional outbox

**Problem it solves:** you need to update your database *and* publish an
event about that update atomically, but a DB write and a broker publish are
two separate systems — if you write to the DB and then the process crashes
before publishing, the event is lost forever (or vice versa: you publish
then crash before committing, and consumers react to something that never
actually happened).

**How it works:** instead of publishing directly, write the event into an
`outbox` table in the *same database transaction* as your business data
change — so either both are committed or neither is. A separate process
(a polling publisher, or more commonly a CDC connector, see below) reads new
rows from the outbox table and publishes them to Kafka, then marks/deletes
them. This gives you "exactly-once effect" (the event is guaranteed to be
published if and only if the business change committed) using only the
atomicity your database already gives you for free.

```
[Service] --(1 tx: write Order row + write OutboxEvent row)--> [DB]
                                                                  │
                                                    (2) CDC/poller reads outbox
                                                                  │
                                                                  v
                                                          [Kafka topic]
```

### Change Data Capture (CDC)

**Problem it solves:** you want to publish events reflecting changes to a
database without modifying application code to explicitly emit them
everywhere writes happen (easy to forget one write path), and without
double-writing (write to DB, then separately write to broker — the same
atomicity problem the outbox pattern solves, just at a lower level).

**How it works:** a CDC tool tails the database's transaction/write-ahead
log (e.g., Postgres logical replication / WAL, MySQL binlog) and turns every
row insert/update/delete into an event, published to Kafka. Debezium is the
standard open-source CDC connector for Kafka Connect. Because CDC reads the
log directly, it captures *every* write regardless of which code path made
it, and it pairs naturally with the outbox pattern: CDC tails the `outbox`
table specifically, so you get outbox-pattern atomicity without writing your
own polling publisher.

**CDC vs. outbox, when to use which:**
- Outbox alone (with a polling publisher) — simplest to reason about, no
  extra infra, but polling adds latency and load.
- CDC on the outbox table (outbox + Debezium) — near-real-time, no
  polling, but requires running/operating Kafka Connect + Debezium.
- CDC on business tables directly (no outbox) — simplest to wire up (no
  outbox table needed) but couples your event shape to your DB schema, and
  every row change becomes an event whether or not it's a meaningful
  domain fact — often too low-level/noisy compared to a deliberately
  designed domain event.

### Saga pattern

**Problem it solves:** a business transaction spans multiple services (e.g.,
"place order" = reserve inventory + charge payment + schedule shipping), so
a single ACID transaction across all of them isn't possible — you need a way
to reach eventual consistency and to *undo* completed steps if a later step
fails.

**How it works:** break the transaction into a sequence of local
transactions, each with a **compensating action** that semantically undoes
it (`ReleaseInventory` compensates `ReserveInventory`). Coordinated either
via choreography (each service listens for the previous step's event and
emits its own, including compensation events on failure) or via
orchestration (a saga orchestrator explicitly invokes each step and its
compensation on failure — generally the more maintainable choice once a saga
has more than 2-3 steps).

### Event sourcing

**Problem it solves:** normally you store current state ("order status =
shipped") and lose the history of how you got there. Event sourcing stores
the sequence of events as the source of truth and derives current state by
replaying them.

**How it works:** every state change is appended as an event
(`OrderPlaced`, `OrderPaid`, `OrderShipped`) to an event store; current
state is a projection/fold over that log. Gives you a full audit trail and
lets you rebuild new read models/projections after the fact, but adds real
complexity (versioning events forever, snapshotting for performance,
eventual consistency of projections) — reach for it only when the audit
trail or replayability is itself a requirement, not by default. Often
confused with EDA generally, but they're separable: you can do EDA without
event sourcing (services just publish events about state changes stored
conventionally, as above) and event sourcing without cross-service EDA
(a single service using it internally as its persistence model).

### Competing consumers

**Problem it solves:** one consumer instance can't keep up with the event
volume.

**How it works:** run multiple instances of the same consumer as a group
(a Kafka consumer group, or multiple readers off the same queue) so
partitions/messages are load-balanced across instances — this is built into
Kafka's consumer-group model directly, described above.

---

## When EDA is worth it (and when it isn't)

**Worth it:**
- Multiple, independently owned services need to react to the same fact,
  and new consumers should be addable without touching the producer
  (fan-out).
- Producer and consumer need decoupled availability/throughput — the
  producer shouldn't block or fail if a consumer is slow or down.
- The business process is naturally a sequence of "something happened, then
  something else happened" facts (order lifecycle, payment lifecycle).

**Not worth it:**
- A synchronous request truly needs an immediate answer from one specific
  service — forcing that through events (publish, wait, poll for a reply
  event) is slower and more complex than a direct call for no benefit.
- Small systems with a single consumer per event, where the ordering/
  idempotency/schema-evolution overhead of a broker isn't buying you
  anything a direct call and a retry wouldn't.
