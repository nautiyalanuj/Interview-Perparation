## PostgreSQL vs Mysql
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

## Materialized views
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



