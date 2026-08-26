# Database Isolation Levels

Isolation controls how concurrent transactions see each other's changes.

## Common Concurrency Problems

### 1. Dirty Read

Transaction A updates a value but does not commit:

```sql
UPDATE account
SET balance = 500;
```

Transaction B reads the value before A commits.

If Transaction A later rolls back, Transaction B has read invalid data.

### 2. Non-Repeatable Read

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

### 3. Phantom Read

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

---

# Isolation Levels

## 1. Read Uncommitted

Lowest isolation level.

Allows:

- Dirty Reads
- Non-Repeatable Reads
- Phantom Reads

Rarely used in production systems.

---

## 2. Read Committed

Default isolation level in PostgreSQL.

Prevents:

- Dirty Reads

Allows:

- Non-Repeatable Reads
- Phantom Reads

Example:

Only committed data can be read. If another transaction commits changes while the current transaction is running, later queries may observe those changes.

---

## 3. Repeatable Read

Prevents:

- Dirty Reads
- Non-Repeatable Reads

PostgreSQL uses MVCC snapshots, allowing a transaction to see a consistent snapshot of the database.

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

Highest isolation level.

Prevents:

- Dirty Reads
- Non-Repeatable Reads
- Phantom Reads

The database guarantees a result equivalent to transactions running one after another.

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

# Summary Table

| Isolation Level | Dirty Read | Non-Repeatable Read | Phantom Read |
|----------------|------------|---------------------|--------------|
| Read Uncommitted | ✅ | ✅ | ✅ |
| Read Committed | ❌ | ✅ | ✅ |
| Repeatable Read | ❌ | ❌ | Mostly ❌ (PostgreSQL prevents through MVCC snapshots) |
| Serializable | ❌ | ❌ | ❌ |

## Interview Summary

- **Materialized View**: A precomputed query result stored on disk for faster reads.
- **PostgreSQL vs MySQL**: PostgreSQL provides advanced SQL features, JSONB support, extensibility, and robust MVCC implementation.
- **Isolation Level**: Defines how transactions interact and which concurrency anomalies are allowed.
