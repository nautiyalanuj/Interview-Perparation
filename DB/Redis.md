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
- ***As distributed lock**
  - Used while booking the tickets
    
   ``` SET lock my-token NX EX 30 ```

   Set the lock with my-token basically who owns the lock for 30 second if that key doesn't exist
    - It is a pessimistic locking
    - Redis also supports optimistic concurrency control: WATCH a key, run your transaction with MULTI/EXEC, and the transaction aborts if the watched key changed in the meantime.
    
- Redis for Leaderboards
  - Redis' sorted sets maintain ordered data that can be queried in log time, making them a natural fit for leaderboard applications. The high write throughput and low read latency make this especially useful for scaled applications where something like a SQL DB will start to struggle.
- Redis for Rate Limiting
- Redis for Proximity Search
- As pub-sub
  - Socket.io uses same for maintaining groups.
 
## Distributed Lock problem
- Master-Replica Failover
  - Client A acquires a lock on the master node. Before the master can sync this lock write to the replica, the master crashes. The replica is promoted to the new master, but it does not have Client A's lock. Client B can now acquire the same lock, violating mutual exclusion.
  - Not present in single node
- Long Process Pauses (GC or Stop-the-World)
  -  Client A acquires the lock with a 10-second TTL. Then, Client A hits a long Garbage Collection (GC) pause or network freeze for 15 seconds. Because the TTL expires, Redis automatically releases the lock. Client B acquires the lock and starts modifying the resource. When Client A wakes up, it still thinks it owns the lock and continues modifying the resource concurrently.
- Clock Drift and NTP Jumps
  - If a system administrator manually changes a server time, or an NTP daemon performs a sudden clock jump forward or backward, lock expiration times become inaccurate. Nodes may expire locks much faster or slower than expected, allowing multiple clients to hold the same lock simultaneously
  - Again not a problem with sinle node 
- Complete Lack of Fencing Tokens
  - In true consensus systems (like Apache ZooKeeper or etcd), every time a client acquires a lock, the system issues a fencing token (a monotonically increasing number that goes up with every lock grant)
  - When a client sends a write request to a database, the database checks the token. If a delayed client attempts a write with an old token, the database rejects it.
  - But redis cannot inherently generate fencing tokens.
- **WATCH** is excellent when Redis is your database, rather than a lock manager for an external database. It ensures that a value inside Redis hasn't changed between the time you read it and the time you write it back. But it cans solve the GC Pause Problem
  - It Doesn't Solve Master-Replica Failover
  - WATCH / MULTI: Solves atomicity inside Redis for a single client connection.
  - Distributed Locks: Solves mutual exclusion for external resources (like databases or third-party APIs). WATCH cannot bridge this gap.
  - If Client A watches the lock key, checks it, and then goes into a GC pause, the lock expires. Client B creates the lock. When Client A wakes up and tries to modify the lock key via a transaction (EXEC), Redis will abort Client A's transaction.
    - The Catch: While WATCH stopped Client A from modifying the Redis key, it did not stop Client A from executing its database write query. The database write happened during the pause, before Client A ever reached the EXEC command.

|Problem| Does it exist in Single Instance?| Why?|
|-|-|-|
|Master-Replica Failover| ❌ No| No replicas exist to cause split-brain.|
|Clock Drift| ❌ No | Only one local clock is used to count TTL.|
|GC / Process Pauses| ⚠️ Yes| The client freezes after getting the lock, blinding it to expiration.|
|Lack of Fencing|⚠️ Yes| Redis still cannot enforce order on downstream databases.|
|Availability SPOF|⚠️ Yes|If the single node goes down, your locking system breaks entirely.|


### Then why to use it as distributed lock?
- It remains popular because not every application requires mathematical perfection, and Redis offers an exceptional trade-off between performance, simplicity, and "good-enough" reliability.
- Preventing two cron jobs from parsing the same massive CSV file at the same time. If the lock fails, the worst-case scenario is wasted CPU cycles.
- A cache stampede protection layer. If 10,000 users request a trending article simultaneously, you lock the backend generation so only one query hits the database while the other 9,999 wait. If the lock fails, your database takes a minor hit, but no data is corrupted.
- Your Operations are Naturally Idempotent, so even if lock fails no damage done.

### When not to use it
- Financial Transactions (e.g., Booking a hotel seat, transferring money, inventory checkout counters)
- Stateful Cluster Coordination (e.g., Electing a leader node to manage a distributed cluster or database shards)
