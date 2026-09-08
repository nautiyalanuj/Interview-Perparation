# Interview
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
    - erializable Isolation (SSI): At the SERIALIZABLE level, Postgres uses a fully optimistic approach called Serializable Snapshot Isolation. It lets transactions execute completely without blocking, checks for conflicts right before committing, and aborts the transaction if a write skew is found
- The Pessimistic Reality: Row Updates (Write vs. Write)
  - When two transactions try to update or delete the exact same row at the same time, Postgres shifts to a pessimistic strategy
    - Implicit Row Locks: Postgres will automatically place a strict, pessimistic lock on that specific row.
    - Blocking: The second transaction does not fail or abort immediately. Instead, it is forced to pause and wait (block) until the first transaction either commits or rolls back   
