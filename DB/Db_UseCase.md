- Infrastructure metrics
  -  Prometheus
  -  InfluxDB
  -  TimescaleDB
- Vehicle tracking / GPS history
  - TimescaleDB
  - InfluxDB
  - Cassandra
- Real-time nearby searches
  - Redis GEO
  - PostGIS
  - Elasticsearch
- Uber
  - Users/Trips => MySQL or Distributed SQL
  - Location History => Cassandra-like storage
  - Current Locations => Redis / Geospatial Services
  - Metrics => Prometheus
  - Logs => Elasticsearch/OpenSearch
  - Analytics => Data Lake + Spark
- What interviewers want
```
Need transactions?
→ Postgres

Need search?
→ Elasticsearch

Need metrics?
→ Prometheus

Need nearest drivers?
→ Geospatial index

Need billions of events?
→ Cassandra or ClickHouse

Need caching?
→ Redis
```
