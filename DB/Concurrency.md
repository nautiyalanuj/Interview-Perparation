# 1. Interview
- Just check [Concurrency](https://www.hellointerview.com/learn/courses/system-design/lesson/contention/dealing-with-contention) page in hellointerview

# 2. Concurrency Control
- **When does db actually lock the row, at start of transaction or basically whenever it find some write needs to be done or at the end when it need to commit and start taking lock**
  - Deferring all row locking until COMMIT time is actually a real database design strategy called Optimistic Concurrency Control (OCC), whereas engines like PostgreSQL and InnoDB use Pessimistic Concurrency Control (PCC).
  - Instead of locking rows to prevent conflicts during execution, OCC defers all checks to a validation phase at COMMIT and relies on isolated workspaces.

---

### A. How OCC Prevents Dirty Writes and Corrupted State
- **Dirty Writes**: Transaction B could modify the exact same row half a second later, overwriting Transaction A's uncommitted work mid-transaction.
- **Corrupted State**: Subsequent SQL queries within Transaction A would see inconsistent state or invalid calculations because another transaction altered the row mid-flight.

**Pessimistic approach:** Locks rows live in shared storage so no one else can touch them while mid-transaction.

**OCC approach:** Transactions never modify shared production data while running.

* **Private Workspace (Shadow Copies):** When a transaction runs `UPDATE` or `DELETE`, it writes changes to a **private, isolated buffer** (local memory or a temporary snapshot).
* **Zero Production Impact:** The shared database state remains untouched during statement execution, so Dirty Writes cannot happen—other transactions simply don't see or touch these pending changes.
* **Atomic Validation at Commit:** At `COMMIT`, OCC enters a brief, atomic validation step. If no other transaction modified those same rows while this transaction was running, the private workspace changes are flushed to the actual database all at once.

---

### B. How OCC Handles "Wasted Work" and Collisions

**Pessimistic approach:** Forces transactions to wait/block early so work isn't wasted at the end.

**OCC approach:** Accepts the risk of wasted work as a trade-off for zero locking overhead.

* **The Optimistic Premise:** OCC assumes that **conflicts are rare** (e.g., millions of users editing *different* profile pages or shopping items). Under low contention, 99%+ of transactions validate successfully on the first try without ever waiting on a lock.
* **Abort and Retry:** If a conflict *does* occur (Transaction B committed an update to row `100` while Transaction A was still working on its private buffer), Transaction A fails validation at `COMMIT`.
* **The Trade-Off:** Transaction A’s work is discarded and rolled back. The application receives a serialization error and simply retries the entire transaction. You trade CPU/time on rare retries to get massive throughput gains when collisions are low.

---

### C. How OCC Ensures Read Consistency Within the Same Transaction
- In standard SQL, a transaction expects to see its own updates. If you run:
```
BEGIN;
UPDATE inventory SET stock = stock - 1 WHERE id = 100; -- Executed at T1
SELECT stock FROM inventory WHERE id = 100;            -- Executed at T2
```
- At T2, the SELECT query must return the updated stock value. To guarantee that no other transaction interferes with row 100 between T1 and COMMIT, the row must be locked immediately at T1.

**Pessimistic approach:** Locks the row so it stays unchanged between `UPDATE` and subsequent `SELECT` calls.

**OCC approach:** Reads from a combination of the base snapshot and the transaction's own local buffer.

* **Local Read Merging:** When a transaction executes `UPDATE inventory SET stock = stock - 1` followed by `SELECT stock`, the database engine queries its **private workspace first**.
* **Self-Consistency:** It sees its own uncommitted changes in local memory and returns the modified `stock` value to the application code.
* **Snapshot Isolation:** For all other unmodified rows, it reads from a point-in-time snapshot of the database created when the transaction started.

---

### Summary Comparison

| Problem | Pessimistic Approach (PostgreSQL / MySQL) | Optimistic Approach (DynamoDB / Custom App Logic) |
| --- | --- | --- |
| **Data Modifications** | Modifies shared data in place **immediately** (requires row locks). | Modifies a **private local workspace** (no locks needed). |
| **Handling Conflicts** | **Wait early:** Blocks transactions when lock collisions occur mid-flight. | **Abort late:** Aborts and retries at `COMMIT` if data changed concurrently. |
| **Read Consistency** | Holds row locks to prevent concurrent updates. | Merges base database snapshot with local uncommitted buffer. |
| **Best Workload** | **High Contention:** Frequent updates to the same rows (e.g., inventory counts, bank balances). | **Low Contention:** Mostly reads, or updates to distinct rows (e.g., user profiles, document editing). |


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


# 3. Fencing Token (Distributed Lock Problem)
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


 
# 4. Common Problems

|Isolation Level| Dirty Reads | Non-Repeatable Reads| Phantom Reads| Write Skew|
|-|-|-|-|-|
|Read Uncommitted|⚠️ Allowed|⚠️ Allowed|⚠️ Allowed|⚠️ Allowed|
|Read Committed|✅ Solved|⚠️ Allowed|⚠️ Allowed|⚠️ Allowed|
|Repeatable Read|✅ Solved|✅ Solved|✅ Solved (In Postgres through MVCC snapshots)|⚠️ Allowed|
|Serializable|✅ Solved|✅ Solved|✅ Solved|✅ Solved|

## Dirty Read

Transaction A updates a value but does not commit:

```sql
UPDATE account
SET balance = 500;
```

Transaction B reads the value before A commits.

If Transaction A later rolls back, Transaction B has read invalid data.

## Non-Repeatable Read

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

## Phantom Read

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

# 5. Isolation Levels
- Isolation Levels define the rules of what data a transaction can see;
- Isolation level determines how transactions interact with each other's uncommitted or committed changes.
- The specific property that stops concurrency issues (classic bank issue) is Isolation level and is solved in serializable.
- Can we define isolation factor for a transaction in SQl db like postgres or mysql? SO, we can have repeatable-read isloation level at db level and for transaction we use serailizable and vice-versa serializable at db and transaction using repeatable-read?
   - Yes, you can set a default isolation level globally at the database level and then override it for individual transactions using SET TRANSACTION ISOLATION LEVEL or when beginning a transaction block.

## Read Uncommitted

- Lowest isolation level.
- Allows:
  - Dirty Reads
  - Non-Repeatable Reads
  - Phantom Reads
- Rarely used in production systems.

---

## Read Committed

- Default isolation level in PostgreSQL.
- Prevents:
  - Dirty Reads
- Allows:
  - Non-Repeatable Reads
  - Phantom Reads

Example:

Only committed data can be read. If another transaction commits changes while the current transaction is running, later queries may observe those changes.

---

## Repeatable Read

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

## Serializable

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

