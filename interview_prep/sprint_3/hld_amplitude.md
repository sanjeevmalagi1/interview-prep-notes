Amplitude is a data/event tracking platform

## Requirements

### Functional Requirements
- Users should be able to record / store the events with properties
- Users should be able to query the stored events ()
- Multi Tenant system (multiple organizations can operate)

### Non Functional Requirements
- System need to be highly available (Availability >> Consistency)
- Massive Scale (million of requests per second)
- Low Latency ingestion : System should response as soon as possible (especially for recording the events)
- Durable storage (no data loss)
- Scalable Querying (OLAP Style)
- Cost affective cold storage

## Entities
- User
  - user_id
  - device_id

- Event 
  - event_id
  - user_id
  - event_name
  - timestamp
  - app_id
  - properties: {
    key: value
  }

- Application
  - app_id
  - api_key

## API Design

### Record Events
POST v1/events
Headers: Authorization: API_KEY
Body:
{
  event_name: "x",
  user_id: "z",
  timestamp: "1710000000",
  properties: {
    key: value
  }
}

Response: 
200 OK

### Query Events
GET /v1/events/query
Query Parameters: {
  "app_id": ,
  "event_name": "x",
  "start_time""
  "end_time":
  filters (JSON)
}

Response:
200
{
  "count": 1000,
  "results": [...]
}

### Aggregation
GET /v1/events/aggregate
Params:
  metric = count | unique_users
  group_by = event_name | day

## High Level Design
### Users should be able to record / store the events with properties

Client/SDK --> API Gateway --> Ingestion Service --> Message Queue (Kafka) --> Queue Processing (Flink/Spark) --> Storage System --> Click House / Druid (hot) --> Query Service --> API --> Dashboard
                          --> S3 (cold)

Ingestion Service
- Stateless services
- Validates events
- Assigns event_id
- Pushes to Kafka

Message Queue (kafka):
- Durable Buffer
- Handles traffic spikes
- Decouples ingestion & processing

Partitioning by app_id or user_id

Stream Processing
- Tools: Flink/ Spark 
- Responsibilities:
    - Enrich events
    - Sessionization
    - Pre-aggregation
    - Filtering bad events

Query Service
- Translates API -> DB queries
- Handles:
  - Filtering
  - Pagination
  - Aggregation

## Potential Deep Dives

### Caching
  - Use Redis
  - Cache most frequently used queries

### Scaling Strategy
  - Horizontal scaling of ingestion service
  - Kafka partitions scale throughput
  - Batch writes to storage

### Query Scaling
  - OLAP DB (like ClickHouse)
  - Pre Aggregation (materialize views)

### Data modeling on ingestion
  - Wide Column db like 
  - Flexible Schema
  - Easy to write

### Event streaming
  - Exactly-Once vs At-Least-Once 
  - Kafka uses At least once
  - dedupe using: event_id-timestamp
