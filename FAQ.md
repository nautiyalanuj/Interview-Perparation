# Questions
## We say TCP is reliable, what happen if connection lost, does TCP or application level protocol say websocket provide relaibility?
  - TCP reliability is on network, so till connection is maintained TCP will send you packet, but if connection is lost neither TCP nor websocket provide any guarantee. We would have to right our custom logic at application layer for same.
</BR></BR>
## Can kafka be used for buffering and sending message for chat like application with billion of users? i.e. using Kafka as a per-user or per-conversation message queue/inbox for offline delivery?
```
User A (Sends Msg) ──► Chat API ──► Kafka Topic ──► User B Online?
                                                         │
                                    ┌────────────────────┴────────────────────┐
                                    ▼                                         ▼
                             [ YES: Online ]                          [ NO: Offline ]
                   Deliver via WebSocket/Push                Do nothing. When User B opens app,
                                                             service polls Kafka starting from
                                                             User B's last committed offset.
```
 - Answer: it is rarely used
 - Advantages
   - Extreme Write Speed & Buffer: Messages are instantly written to Kafka's disk-backed append log. Senders get immediate delivery confirmation without waiting for receivers to get the message.
   - Built-in Replayability: Kafka natively supports consumer offsets. You don't need a custom tracking mechanism for "what was the last message ID this user fetched" because Kafka tracks offsets out-of-the-box.
   - No Database Writes on High-Throughput Paths: Bypassing a database on write path reduces immediate disk I/O bottlenecks during massive peak load spikes.
- Disadvantages & Why It Fails at Scale
  - The Partition & Topic Explosion (Major Bottleneck) 
    - Option A: One topic per user/conversation. Having 1 billion topics will crash Kafka's metadata controller (KRaft or ZooKeeper) and consume absurd amounts of memory for file handles. Kafka cannot handle millions of topics.
    - Option B: One consumer group per user on shared topics. If 10 million offline users come online at the same time, you'll create 10 million active consumer groups. Kafka brokers overload trying to track and balance group metadata for that many consumers.
  - Wasteful Disk & Network I/O (The "Filter" Problem) : Wasteful Disk & Network I/O (The "Filter" Problem)
    - When User B comes online and asks for their messages from offset 100,000, the worker must scan millions of records in that partition, throwing away 99.99% of messages that belong to other users just to find the 5 messages intended for User B. This turns O(1) direct inbox lookups into heavy O(N) log scans over network and disk.
  - Out-of-Order Offset Progression
    - If a user is in multiple group chats distributed across different Kafka partitions, tracking a single global "last saved offset" per user is impossible. You would need to store and commit separate offsets per partition per user, creating massive metadata overhead in your database.
  - Data Retention Conflicts
    - Kafka storage costs scale linearly to petabytes for message stored for sending to user
</BR></BR>
## If our data is already stored in PostgreSQL, why can't we use PostgreSQL's built-in full-text search capabilities instead of introducing Elasticsearch? We could use multiple read replicas and perform searches directly against PostgreSQL.
- Yes, that's a very reasonable question, and in many systems PostgreSQL full-text search is absolutely sufficient, especially in the early stages. The answer usually comes down to search requirements and scale, not whether PostgreSQL can search text. For a system like: 1-50 million records, basic keyword search, and no fancy ranking requirements; dedicated read replicas works perfectly.
- Where Elasticsearch Starts Winning:
  - PostgreSQL ranking is relatively basic. Elasticsearch provides:
    - BM25 ranking, Field boosting, Relevance tuning, Synonyms and Typo tolerance
    - Example: Search for: iphone charger
      - Elasticsearch can understand:
        - charger ≈ charging cable
        - iphone 14 charger
        - iphone fast charging adapter
  - Fuzzy Search at Scale
    - Elasticsearch handles: spelling mistakes, approximate matching and phonetic matching much more efficiently. So amzn, mmicrosfot, iphne can be handled in better way.
  - Complex Search Features
    - Autocomplete, Suggestions, Faceted search, Aggregations, Geospatial queries, "Did you mean?", Highlighting matched text. Building these in PostgreSQL can become difficult and inefficient.
  - Huge Volumes of Text
    - Imagine: Millions of products, Billions of logs, Large document repositories. Elasticsearch is designed as a distributed inverted-index engine. Searching billions of documents is its primary purpose. PostgreSQL is still fundamentally a transactional database.
  - Isolation from OLTP Workloads
    - Suppose you have: 500 GB DB and 150 GB search index.
      - With Elasticsearch, search data can be much more compactly organised for text retrieval. and no replication of whole data.
      - With PostgreSQL every replica carries: table data, search indexes and transaction metadata.
        ```
        Primary 650 GB
        Replica 1 650 GB
        Replica 2 650 GB
        Replica 3 650 GB
        Total: ~2.6 TB
        ```
    - Suppose tomorrow searches increase 50x.
      - Again same problem, we will add more replicas, but Elasticsearch was designed specifically for this scaling model.
    - Replication Costs
      - Every write must be replicated to every PostgreSQL replica. Imagine:  10k writes/sec. Now every search replica receives all WAL changes, including data irrelevant to search. Elasticsearch can consume only the fields necessary for indexing.
</BR></BR>

## Can we use a Time Series DB for real-time analytics instead of using an OLAP system? Time Series DBs are highly optimised for ingesting millions of log events and efficiently running aggregation queries, similar to how they're used for monitoring and metrics. (Example Ad-click aggregator system)
- Flink + OLAP is usually chosen because the problem is not just real-time aggregation. It's real-time aggregation + **flexible analytics**. A TSDB excels at the first part but often struggles with the second at large scale.
- Another major issue: **Cardinality explosion**
  - This is probably the biggest reason. Imagine dimensions
    ```
      campaignId     = 1M
      country        = 200
      device         = 20
      browser        = 50
      ageGroup       = 10
    ```
  - Possible combinations:1M × 200 × 20 × 50 × 10 = 20 trillion combinations
  - TSDBs maintain indexes on dimensions (tags). Very high-cardinality dimensions can become expensive. Ad-tech data is notorious for having massive cardinality:
- **Flexible analytics**: Suppose every click event looks like 
  ```
  {
    "timestamp": "...",
    "advertiserId": 123,
    "campaignId": 456,
    "creativeId": 789,
    "country": "IN",
    "device": "Android",
    "browser": "Chrome",
    "ageGroup": "18-24",
    "clickCost": 0.25
  }
  ```
  - An advertiser may ask:
    - Clicks per minute for last hour ✅
    - CTR by country for today ✅
    - Top campaigns in last 7 days ✅
    - These are good TSDB workloads.
  - But then business users ask:
    - Compare CTR of Android vs iOS users in India who clicked after 6 PM.
    - Join click data with campaign metadata.
    - Join with billing data.
    - Calculate advertiser spend across all campaigns grouped by industry.
    - Run a dashboard with 20 filters.
    - This starts looking more like a warehouse problem than a metrics problem.
- Why monitoring systems can use TSDB?
  - Monitoring data is relatively predictable. For example: cpu_usage, memory_usage, request_count, latency_p99.
  - Queries are usually: avg(cpu_usage) by host, sum(request_count) over time.
  - Limited dimensions, no complex joins. That's why Prometheus, VictoriaMetrics, InfluxDB etc. work so well.
 - **(What about startups?)** - Since the product is early-stage and scale is uncertain, I would start with PostgreSQL + TimescaleDB(postgressql) because it keeps the architecture simple, supports high write throughput, time-based analytics, and real-time aggregations. I would defer Kafka, Flink, and a dedicated OLAP store until there is evidence that Postgres can no longer meet latency, throughput, or analytical requirements. This follows the principle of evolving architecture with business growth rather than designing for hypothetical scale.
   - _Why it works initially?_
     - Even 100M clicks/day is ~1,157 clicks/sec. A properly tuned Postgres instance can still sustain that. Its growth can be like, first introduce Aggregated tables
      ``` 
       Postgres
        |
        +--> Raw events
        |
        +--> Aggregated tables
      ```
       - For example:
         ```
         campaign_hourly_stats
          ---------------------
          hour
          campaign_id
          clicks
          impressions
          revenue
         ```
        - Update these tables continuously and query on same?
    - Then may be based on bottlenecks keeps on evolving your database.
