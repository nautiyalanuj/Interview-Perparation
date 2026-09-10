# Limits
 - Tables start getting unwieldy past 100M rows
 - Full-text search works well up to tens of millions of documents
 - Complex joins become challenging with tables >10M rows
 - Performance drops significantly when working set exceeds available RAM
 - Simple inserts: ~5,000 per second per core
 - Updates with index modifications: ~1,000-2,000 per second per core
 - Complex transactions (multiple tables/indexes): Hundreds per second
 - Simple indexed lookups: tens of thousands per second per core (often 50k+)
 - Multi-table joins with indexes: thousands to tens of thousands per second

# PostgreSQL vs Mysql
- Better SQL Standards Support
- More Advanced Query Optimizer
  - Postgres Handle complex query better
  - For analytics-heavy workloads, Postgres often performs better.
- Rich Data Types
  - Arrays
   ```
   CREATE TABLE users (
   tags TEXT[]
   );
   ```
  - JSONB
    ```
    CREATE TABLE events (
    data JSONB
    );
    ```
    Query JSON directly:
    ```
    SELECT *
    FROM events
    WHERE data->>'city' = 'Delhi';
    ```
- Strong Concurrency (MVCC)
  - Postgres has excellent implementation of MVCC where
    - Readers do not block writers.
    - Writers minimally affect readers.
 
- Extensibility
  - Custom data types, functions, operators and extensions
    - Support text search using GIN and tsvector
    - Support json using GIN and jsonb
    - Support Geospatial Search with PostGIS
    - Support vector search for AI.
    - Support Timeseries extension which is TimescaleDB  => PostgreSQL + Time-series extensions
      -  Alternative for prometheus
      -  Full SQL support
      -  Joins
      -  Relational data + metrics together
      -  Works excellently with Grafana.

# Other
- PostgreSQL forks a new OS process for each connection, so hundreds of connections consume significant memory and CPU for context switching. In practice, you'll want a connection pooler (like PgBouncer) in front of PostgreSQL to multiplex application connections onto a smaller pool of database connections. This is especially important when running many application instances.
- Partition
  - For large tables, partitioning can improve both read and write performance by splitting data across multiple physical tables. The most common use case is time-based partitioning.
  - Why does this help writes? First, different database sessions can write to different partitions simultaneously, increasing concurrency. Second, when data is inserted, index updates only need to happen on the relevant partition rather than the entire table.
  - Conveniently, it also helps with reads. When users view recent posts, PostgreSQL only needs to scan the recent partitions. No need to wade through years of historical data.

# Materialized views
- Example
```
CREATE MATERIALIZED VIEW monthly_sales AS
SELECT product_id, SUM(amount)
FROM sales
GROUP BY product_id;
```
So, whenever someone run query we do not re-execute the aggregation and simply returns precomputed data.
- Advantage
  - As it's a pre-computed view,  query is very fast
  - Real-world use cases include Dashboards, Analytics, Reporting systems, Aggregated metrics and Data warehouse workloads
- Disadvantage
  - Its a stale data.



