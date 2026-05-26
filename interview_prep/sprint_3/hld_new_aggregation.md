News Aggregator System Design

## Requirements

### Functional Requirements
- New Ingestion
  - Scrape / Crawl articles from N number of sources
  - Support RSS Feed
  - Handle format / rules per publisher

- Content Processing
  - Extract 
    - Title 
    - Body
    - Author
    - Timestamp
    - Images
  - Deduplicate similar news across sources
  - Categorize into topic (Politics, Tech, Sports, etc.)
  - Rank news (trending, relevance)

- User Feed
  - Infinite Scroll Feed
  - Personalization 
  - Trending / Global feed

- Filtering 
  - By category, source, region

- Near real time updates
  - News articles appear quickly enough (a bit slow is fine)

### Non Functional Requirements
  - Scalability 
    - (millions of articles per day)
    - Thousands of news sources
  - Availability
  - Low Latency feed
  - Freshness 
  - Fault Tolerance (scrapers fail / system still works)
  - Extensibility 
    - Easy to add new sources / format / rules


## Entities
- Article
  - id
  - title
  - content
  - source_id
  - author
  - published_at
  - category
  - language
  - url
  - image_url

- Source
  - id
  - name
  - base_url
  - rss_url
  - crawl_config (selector, parsing rules)

- Topic
  - id
  - name

User (optional)
  - id
  - preferences
  - history

Cluster (for deduplication)
- id
- canonical_article_id
- similar_articles[]

## API Design

- Get Feed (Infinite Scroll)

GET /v1/feed?cursor=<cursor>&limit=20&category=tech
Response:
{
  "articles": [...],
  "next_cursor": "abc123"
}

- Get Article Details
GET /v1/article/{article_id}

- Trending articles
GET /v1/trending


## High Level Design

News Source -> Crawlers -> Message Queue (Kafka) -> Pipeline -> Search Index -> API -> Clients
                                                             -> Feed Storage (Cassadra / Redis)
                                                             
### News Ingestion (Massive Scale)
  Approach: Hybrid ingestion
  - For RSS Feed
    - Efficient, structured
    - Pull every few seconds
  - Web Crawlers
    - For sites without RSS
    - Use distributed crawling

### Message Queue (Decoupling)
  - Topics:
    raw_articles
    parsed_articles
    deduplicated_articles

  - Benefits:
    Backpressure handling
    Replay capability
    Loose coupling
### Processing Pipeline
  - Parsing
  - Deduplication
  - Classification
  - Ranking

### Storage Design
  - Feed Store (Hot Path)
    - Cassandra / DynamoDB (write-heavy)
    - Redis (for caching top feeds)
    Data model:

    feed:{category} -> sorted list of article_ids
    Sorted by:
    timestamp / score

  - Search Index


## Potential Deep Dives

  1. Scalability

    Problem:

    Millions of articles/day

    Solution:

    Horizontal scaling:
    Crawlers → autoscale
    Kafka → partitioned
    Processing → consumer groups

  2. Low Latency Feed

  Optimizations:

  Precompute feeds
  Cache hot categories in Redis
  Use CDN for global users

  6. Deduplication at Scale (Hard Problem ⚠️)
    Use:
    SimHash for fast detection
    Vector DB (FAISS) for semantic similarity

    