# Introduction

- Redis is really, really fast. A single Redis node can handle something like 100k writes per second. The command itself executes in microseconds; over the network you'll see sub-millisecond reads
- Data structure supported by redis
  - Strings, sets, hashes, lists, sorted-sets, streams(append-only) and Geospatial Indexes.
  - Redis 8 also ships probabilistic structures like Bloom filters, JSON, and time series support in core
- In addition to simple data structures, Redis also supports different communication patterns like Pub/Sub and Streams
- Persistence : One important reason you might not want to use Redis is because you need durability. Redis has two persistence modes: RDB takes periodic snapshots (a crash loses everything since the last one), and AOF logs every write but only fsyncs once per second by default (a crash loses up to a second of acknowledged writes). You can configure AOF to fsync on every write, but few deployments do because it costs the speed they came for.
- Redis can run as a single node, with a high availability (HA) replica, or as a cluster. When operating as a cluster, every key hashes to one of 16,384 "hash slots", and each slot is assigned to a node.
- Redis clients cache the slot-to-node mapping, so they can compute the slot from the key and connect directly to the node which contains the data they are requesting. When you add a node, slots (and the keys in them) migrate to it.
- Nodes share cluster state with each other (via a gossip protocol), so every node knows the full slot map. If you ask the wrong node for a key, you get a MOVED reply pointing at the right one. The node won't forward the request for you, so clients cache the map and aim for the correct node on the first try.
- Redis replication is asynchronous. The primary acknowledges your write before the replica has seen it, so when a primary dies and a replica is promoted, the last moments of acknowledged writes can simply vanish.
- When two keys need to live together, hash tags make it happen. Only the part of the key inside {braces} gets hashed, so {user:123}:posts and {user:123}:likes always land in the same slot, ready for a MULTI transaction across both.
- In cluster mode, a client issues READONLY on its connection to allow replica reads, and most client libraries expose this as a flag.

# Use-case

- As Cache
- As distributed lock
  - Used while booking the tickets
    
   ``` SET lock my-token NX EX 30 ```

   Set the lock with my-token basically who owns the lock for 30 second if that key doesn't exist
    - It is a pessimistic locking
    - Redis also supports optimistic concurrency control: WATCH a key, run your transaction with MULTI/EXEC, and the transaction aborts if the watched key changed in the meantime.
    
- As pub-sub
  - Socket.io uses same for maintaining groups.  
