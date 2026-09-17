# Interview -
- Just check [Concurrency](https://www.hellointerview.com/learn/courses/system-design/lesson/contention/dealing-with-contention) page in hellointerview

# Isolation level
- Isolation Levels define the rules of what data a transaction can see;
- Isolation level determines how transactions interact with each other's uncommitted or committed changes.
- The specific property that stops concurrency issues (classic bank issue) is Isolation level and is solved in serializable.

  
# Common Problems

|Isolation Level| Dirty Reads | Non-Repeatable Reads| Phantom Reads| Write Skew|
|-|-|-|-|-|
|Read Uncommitted|⚠️ Allowed|⚠️ Allowed|⚠️ Allowed|⚠️ Allowed|
|Read Committed|✅ Solved|⚠️ Allowed|⚠️ Allowed|⚠️ Allowed|
|Repeatable Read|✅ Solved|✅ Solved|✅ Solved (In Postgres through MVCC snapshots)|⚠️ Allowed|
|Serializable|✅ Solved|✅ Solved|✅ Solved|✅ Solved|

## 1. Dirty Read

Transaction A updates a value but does not commit:

```sql
UPDATE account
SET balance = 500;
```

Transaction B reads the value before A commits.

If Transaction A later rolls back, Transaction B has read invalid data.

## 2. Non-Repeatable Read

Transaction A reads a row:

```sql
SELECT balance FROM account;
```

Result:

```text
1000
```

Transaction B updates and commits:

```sql
UPDATE account
SET balance = 1200;
COMMIT;
```

Transaction A reads again:

```sql
SELECT balance FROM account;
```

Result:

```text
1200
```

The same query returned a different result within the same transaction.

## 3. Phantom Read

Transaction A executes:

```sql
SELECT COUNT(*)
FROM orders
WHERE amount > 1000;
```

Result:

```text
10 rows
```

Transaction B inserts a matching row and commits.

Transaction A runs the query again and gets:

```text
11 rows
```

A new row has appeared, creating a phantom read.

## Write Skew 
- A write skew anomaly happens when two concurrent transactions read the same data, calculate overlapping rules, and modify different rows. Neither transaction sees the other's changes, breaking a global system rule.
- Example
  - Imagine a hospital rule: "At least one doctor must remain active on call."Doctors Alice and Bob are currently on call. Both try to check out at the exact same time.
    ``` SELECT COUNT(*) FROM shifts WHERE status = 'active'; -- Returns 2 for both ```

---

# Isolation Levels

## 1. Read Uncommitted

- Lowest isolation level.
- Allows:
  - Dirty Reads
  - Non-Repeatable Reads
  - Phantom Reads
- Rarely used in production systems.

---

## 2. Read Committed

- Default isolation level in PostgreSQL.
- Prevents:
  - Dirty Reads
- Allows:
  - Non-Repeatable Reads
  - Phantom Reads

Example:

Only committed data can be read. If another transaction commits changes while the current transaction is running, later queries may observe those changes.

---

## 3. Repeatable Read

- Prevents:
 - Dirty Reads
 - Non-Repeatable Reads
- PostgreSQL uses MVCC snapshots, allowing a transaction to see a consistent snapshot of the database.

Example:

```sql
BEGIN;
SELECT balance; -- 1000

-- Another transaction updates balance to 2000 and commits

SELECT balance; -- still 1000
COMMIT;
```

---

## 4. Serializable

- Highest isolation level.
- Prevents:
  - Dirty Reads
  - Non-Repeatable Reads
  - Phantom Reads
- The database guarantees a result equivalent to transactions running one after another.

Example:

```text
T1 then T2
```

or

```text
T2 then T1
```

If a serialization conflict occurs, one transaction may be rolled back and must be retried.

---

# Locking/ Concurrency Control

## Pessimistic Locking
- Pessimistic concurrency control assumes that conflicts between transactions are likely. So, before a transaction performs operations on data, it acquires locks to prevent other transactions from accessing the same data in a conflicting way.
- Pessimistic concurrency control is preferred when conflicts are likely, when retrying failed work would be expensive, and when strong correctness and consistency are critical.
- If a transaction wants to read data, it may acquire a shared lock.
- If a transaction wants to modify data, it acquires an exclusive lock.


## Optimistic Locking
- Optimistic concurrency control is preferred when conflicts are rare, contention is low, and occasional retries are acceptable.
- It works well for read-heavy systems, low-conflict business workflows, and application-level updates where blocking transactions upfront would unnecessarily reduce concurrency.

## PostGres
- PostgreSQL uses an optimistic hybrid model. By default, it leans heavily towards optimistic principles using Multi-Version Concurrency Control (MVCC), but it switches to pessimistic mechanics when handling concurrent row writes.
- The Optimistic Foundation: MVCC (Reads vs. Writes)
  - For standard operations, PostgreSQL operates optimistically. It assumes conflicts are rare and tries to avoid locking the database.
    - No Read Locks: Readers never block writers, and writers never block readers. [1] (https://vladmihalcea.com/optimistic-vs-pessimistic-locking/)
    - Row Versioning: When a row is updated, Postgres does not overwrite it or lock out readers. It creates a new version of the row. Concurrent transactions simply read older, committed versions.
    - Serializable Isolation (SSI): At the SERIALIZABLE level, Postgres uses a fully optimistic approach called Serializable Snapshot Isolation. It lets transactions execute completely without blocking, checks for conflicts right before committing, and aborts the transaction if a write skew is found
- The Pessimistic Reality: Row Updates (Write vs. Write)
  - When two transactions try to update or delete the exact same row at the same time, Postgres shifts to a pessimistic strategy
    - Implicit Row Locks: Postgres will automatically place a strict, pessimistic lock on that specific row.
    - Blocking: The second transaction does not fail or abort immediately. Instead, it is forced to pause and wait (block) until the first transaction either commits or rolls back

# Fencing Token
- The problem : Without fencing tokens, distributed locks fail when a client experiences a garbage collection (GC) pause, network lag, or process freeze.
```
Client 1                   Lock Service (etcd/ZooKeeper)              Storage (Database)
   │                                     │                                     │
   ├─── 1. Acquire Lock (Lease: 10s) ───►│                                     │
   │◄── 2. Lock Granted ─────────────────┤                                     │
   │                                     │                                     │
   █ [ Long GC Pause / Freeze ]          │                                     │
   █ (Lease expires on server)          │                                     │
   │                                     │                                     │
   │                     Client 2        │                                     │
   │                        ├─── 3. Acquire Lock ────────────────────────────►│
   │                        │◄── 4. Lock Granted ────────────────────────────┤
   │                        ├─── 5. Write Data ────────────────────────────────────────────────►│ (Succeeds)
   │                                     │                                     │
   │ (Wakes up thinking it still         │                                     │
   │  holds the valid lock)              │                                     │
   ├─── 6. Write Data ─────────────────────────────────────────────────────────────────────────►│ (Corrupts Data!)
```

- The Solution:
  - To prevent stale writes, the lock service increments a counter on every lock grant and attaches it to the lock response. When writing data to downstream storage, the client must pass this token along. The storage engine tracks the highest token it has processed and rejects any write with an older token.**** 
```
Client 1                   Lock Service                              Storage (Database)
   │                                     │                                     [Max Token: 30]
   ├─── 1. Acquire Lock ────────────────►│                                     │
   │◄── 2. Lock Granted (Token = 31) ────┤                                     │
   │                     Client 1's write (Token = 31): Current max is 32. 31 > 32 is FALSE. Storage rejects the update.                │                                     │
   █ [ GC Pause ]                        │                                     │
   │                     Client 2        │                                     │
   │                        ├─── 3. Acquire Lock ────────────────────────────►│
   │                        │◄── 4. Lock Granted (Token = 32) ───────────────┤
   │                        ├─── 5. Write (Token = 32) ─────────────────────────►│ (Succeeds)
   │                                     │                                     [Max Token: 32]
   │ (Wakes up with stale Token 31)      │                                     │
   ├─── 6. Write (Token = 31) ─────────────────────────────────────────────────►│ ❌ REJECTED!
                                                                               (31 < 32)
```
- A separate distributed lock service (like etcd, ZooKeeper, or Consul) grants a lock along with a strictly increasing number (the fencing token). The client attaches this token to all storage calls. The storage layer keeps track of the highest token it has processed and rejects any request containing an older token.
- How Storage Validates the Token
  - Client 2's write (Token = 32): Current max is 30. 32 > 30 is TRUE. Write succeeds, storage updates last_fencing_token = 32.
  - Client 1's write (Token = 31): Current max is 32. 31 > 32 is FALSE. Storage rejects the update.
```
UPDATE resource_table
SET data = 'new_value', last_fencing_token = 32
WHERE resource_id = 'A1'
  AND 32 > last_fencing_token;
```

## Fencing Token vs OCC

### Use ETag / OCC When:
- Your storage system supports conditional writes natively: PostgreSQL, MySQL, MongoDB, AWS DynamoDB (ConditionExpression), and HTTP APIs (If-Match headers) all handle this natively out of the box.
- Contention is low to moderate: Conflicts are rare, so retrying occasionally in application code is cheap.
- Operations are contained within a single data store: You are updating database rows or API objects directly without external side effects.

### Do NOT Use ETag / OCC When:
- The operation involves non-database side effects: ETag/OCC cannot undo an external API call, a sent email, or physical hardware activation if the conditional write fails at the very end.
- Contention is extremely high: Constant retries under high concurrency cause performance degradation due to high conflict rates. (Pessimistic locking)

### Use Fencing Tokens When:
- You have long-running asynchronous tasks: A background worker processes a 10-minute job and needs to ensure no secondary worker took over and finished first while it was paused.
- You are coordinating across heterogenous systems: Managing access to external file systems (S3/NFS), third-party webhooks, or physical hardware where you cannot perform atomic database checks.
- You need strict leader election: Ensuring only one primary worker executes a task at any given instant.

### Do NOT Use Fencing Tokens When:
- A standard database query or OCC handles the job: Adding ZooKeeper or etcd just to guard a database write adds unnecessary operational complexity and infrastructure overhead.
- Your storage layer cannot validate incoming tokens: If the target system (e.g., a simple S3 file upload without headers or a third-party API) has no mechanism to evaluate incoming_token > highest_seen_token, a fencing token loses its protective guarantee.
