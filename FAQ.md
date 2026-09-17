# Questions
- **We say TCP is reliable, what happen if connection lost, does TCP or application level protocol say websocket provide relaibility?**
  - TCP reliability is on network, so till connection is maintained TCP will send you packet, but if connection is lost neither TCP nor websocket provide any guarantee. We would have to right our custom logic at application layer for same.

- **Can kafka be used for buffering and sending message for chat like application with billion of users? i.e. using Kafka as a per-user or per-conversation message queue/inbox for offline delivery?**
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
