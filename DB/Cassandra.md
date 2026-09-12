# ScyllaDB
- ScyllaDB is a high-performance, drop-in C++ rewrite of Apache Cassandra that delivers 5x to 10x higher throughput and significantly lower tail latencies by eliminating Java Virtual Machine (JVM) garbage collection pauses and utilizing a thread-per-core, shared-nothing architecture. 
- While both databases share the same data model, ring architecture, and use Apache Cassandra Query Language (CQL), choosing ScyllaDB over Cassandra involves distinct trade-offs in performance, operational complexity, and licensing.
- Maximum Hardware Efficiency: ScyllaDB uses a shard-per-core design that pins threads to specific CPU cores and manages its own memory and I/O polling directly, fully saturating modern multi-core processors and NVMe drives.
- Reduced Infrastructure Footprint: Due to its superior throughput per node, workloads that require large clusters in Cassandra can often be run on a fraction of the node count in ScyllaDB, slashing infrastructure and hardware costs.
- Self-Tuning & Automated Operations: ScyllaDB auto-tunes against underlying hardware and includes built-in capabilities like Incremental Compaction Strategies (ICS) and Workload Prioritization, removing the heavy operational burden of manual JVM heap tuning and managing compaction storms.
- **Why discord moved away from cassandra -** https://discord.com/blog/how-discord-stores-trillions-of-messages 

# When to Stick with Apache Cassandra
- Strict Open-Source Requirements: ScyllaDB shifted to a source-available license with limitations for large organizations, whereas Apache Cassandra remains 100% open-source and community-governed.
- Ecosystem and Community Maturity: Cassandra has a massive, decades-old enterprise deployment history, a broader pool of in-house administrative expertise, and deep native integration across old legacy data tools.
- Modern Java Enhancements: Recent versions of Apache Cassandra (such as Cassandra 5.0+) have introduced major performance improvements, modern storage engines, and better predictability, narrowing the gap for standard enterprise workload

# Cassandra
- Optimised for write-heavy workload and append only write
- Use LSM based indexing ( Postgres use B+), basically append only
- Use consistency hashing for adding/deleting node
- Works like mongo where data is present in few nodes and replicated to others
  - Each node add as coordinator and give detail about node which owns same.
  - Again speed depends on if we are writing to single node or multiple replica nodes for availability vs latency.
- Use bloom filter for fast read
  - Check data in disk pages through bloom filter
- Use gossip protocol for checking cluster health
- Partition Key - One or more columns that are used to determine what partition the row is in.
- Clustering Key - Zero or more columns that are used to determine the sorted order of rows in a table. Data ordering is important depending on one's data modeling needs, so Cassandra gives users control over this via the clustering keys.
