# Interview Answer
- I would first scale vertically and keep a single primary database. When read traffic becomes dominant, I'd introduce read replicas. If specific entities become hot and latency-sensitive, I'd place Redis in front of the database for those access patterns. I wouldn't introduce caching before proving that repeated reads are actually the bottleneck.
- Traffic has increased. Should we move to multi-write?
  - Not necessarily. I'd first determine whether the bottleneck is storage, read throughput, write throughput, latency, or availability. For write throughput I'd prefer sharding with single-writer ownership because it's significantly simpler. I'd consider active-active multi-write only when global write latency or regional independence becomes a hard business requirement.

# Introduction
When dealing with services which are global we have multiple choices and in this we will discuss same. Let's see the dimensions in which we can divide same

- Dimension 1: Write ownership
  - Single Writer
  - Multiple Writer 
- Dimension 2: Read Ownership
  - Read from primary
  - Read replicas
  - Regional caches
  - CQRS read models

## Decision Tree
```
    Problem?
    │
    ├── Too many reads
    │      ├── Add Replica
    │      └── Add Cache
    │
    ├── Database too large
    │      └── Shard
    │
    ├── Too many writes
    │      └── Shard
    │
    ├── Users far from primary region
    │      └── Multi-Write candidate
    │
    ├── Region outage cannot stop writes
    │      └── Multi-Write candidate
    │
    └── Collaborative editing
           └── Multi-Write required
```

## Real Progression

```
    Single DB
       ↓
    Read Replica
       ↓
    Cache
       ↓
    Sharding / Partitioning
```


# Single-Write Database 

## Single Db
- First is single db, which handle both read and write. A properly tuned PostgreSQL/MySQL instance can often handle: Thousands of TPS and Millions of rows

## Single Db + Read Replica
- Writes go only to Primary.Reads go to replicas.
- Tradeoff is replication lag i.e. Replica may return old data. Common solution:
  - Writes → Primary
  - Fresh reads after write → Primary
- Why replica before cache?
  - Replica = correct data, while Cache = another system to manage

## Single Db + Cache
 - What problem does cache solve?
   - Latency
   - Hotspot traffic

Replica vs Cache

People often confuse these.

|Aspect	|Read Replica|	Cache|
|-|-|-|
|Contains entire DB|	Yes	|No|
|Query flexibility|	Full SQL|	Usually key lookup|
|Data freshness|	High	|Depends|
|Latency|	Good|	Excellent|
|Consistency|	Better|	Worse|
|Handles hotspots|	Moderate |	Excellent|

# Multi-Write Database
```
Multi-Master with Sharding (Shared-Nothing / Partitioned)
[App Client]
   ├── Writes User A-M ──> [Shard Master Node 1] ── (Replicates to Standbys)
   └── Writes User N-Z ──> [Shard Master Node 2] ── (Replicates to Standbys)
   (* Zero write conflicts because data sets are completely segregated *)

Multi-Write Cluster (Active-Active / Shared-Everything or Fully Replicated)
[App Client]
   ├── Writes User A ────> [Active Node 1] ──┐
   │                                         ├── (Bi-directional Sync / Paxos / Raft)
   └── Writes User A ────> [Active Node 2] ──┘
   (* High risk of write conflicts; requires immediate consensus or conflict resolution *)

```

| Feature| Multi-Master with Sharding| Multi-Write Cluster (Active-Active)|
|-|-|-|
|Data Distribution| **Split into pieces**; each master owns a unique shard.| **Fully replicated**; all nodes hold the entire dataset.|
|Write Conflict Risk| **Zero** (if routing logic is correct). | **High** (simultaneous writes to the same row).|
|Scaling Vector| Scaled horizontally by adding more shards.|Scaled geographically for low-latency local access.|
|Query Complexity|**High** for cross-shard joins or aggregations.|**Low**; any node can answer any query locally.|
|Consistency Model| **Strong consistency** per shard.|**Eventual consistency** or synchronous consensus.|
|Operational Overhead| Complex routing layer and re-sharding management. |Complex replication engine and conflict handlers.|


## Choose Multi-Master Sharding if:
- Your data has a natural boundaries (e.g., multi-tenant B2B apps where customers never share data).
- You require strict ACID transactional compliance without risk of data overwrite or complex conflict resolution.
- Scaling Write Throughput, or solving high CPU
- Large Database Size

## Choose Multi-Write Active-Active if:
- You are building a globally distributed consumer app (e.g., social media feeds, IoT ingestion) where sub-second regional write latency is vital.
- High availability is your absolute priority; the system must survive data center outages with zero downtime.
- Your business logic can tolerate eventual consistency or handles concurrent modification conflicts naturally.

## Question
- Suppose your DB handles: 10k writes/sec and 100k reads/sec and Traffic grows to: 100k writes/sec and 1M reads/sec, Would you add multi-write?
  - No, do sharding 
 
# CQRS
Should reads and writes use the same model/storage?

```
Write Model → Primary DB

Read Model → Replica
→ ElasticSearch
→ Cache
→ Materialized View
```

```
           Write Path
                │
                ▼
         Primary Database
                │
             CDC/Event
                │
┌───────────────┼───────────────┐
▼               ▼               ▼
Search Index    Read Replica    Redis Cache
```

