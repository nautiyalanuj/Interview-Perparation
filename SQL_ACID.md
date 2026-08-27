# Interview Answer

> PostgreSQL achieves atomicity using transactions and Write-Ahead Logging (WAL). Changes are first recorded in WAL, and a transaction is considered successful only after its COMMIT record is durably written. During crash recovery, committed transactions are replayed while uncommitted transactions are discarded, guaranteeing all-or-nothing behavior. Consistency is enforced through schema constraints such as primary keys, foreign keys, unique constraints, check constraints, and triggers, ensuring every committed transaction moves the database from one valid state to another valid state.

---

# ACID Mapping in PostgreSQL

| Property | How PostgreSQL Achieves It |
|-----------|---------------------------|
| Atomicity | Transactions + WAL + COMMIT records + Recovery |
| Consistency | Constraints, Foreign Keys, Checks, Triggers |
| Isolation | MVCC (Multi-Version Concurrency Control) |
| Durability | WAL flushed to disk before COMMIT succeeds |

# Atomicity and Consistency in PostgreSQL

Let's use a simple bank transfer example to understand both concepts.

Initial state:

```text
Account A = 1000
Account B = 500
```

Transfer ₹100 from A to B:

```sql
BEGIN;

UPDATE accounts
SET balance = balance - 100
WHERE id = 'A';

UPDATE accounts
SET balance = balance + 100
WHERE id = 'B';

COMMIT;
```

---

# 1. Atomicity (All or Nothing)

Atomicity guarantees:

> Either all operations in a transaction succeed, or none of them do.

## Scenario: Crash in the Middle

Suppose PostgreSQL executes:

```sql
UPDATE accounts
SET balance = balance - 100
WHERE id = 'A';
```

So the state becomes:

```text
A = 900
B = 500
```

Before the second update executes, the server crashes.

Without atomicity:

```text
A = 900
B = 500
```

₹100 has disappeared.

## How PostgreSQL Prevents This

PostgreSQL records transaction changes in the Write-Ahead Log (WAL).

A transaction is considered committed only when a `COMMIT` record is written and flushed to WAL.

If PostgreSQL restarts and finds:

```text
TX123:
  UPDATE A

(No COMMIT record)
```

The transaction is treated as incomplete and rolled back.

Final state:

```text
A = 1000
B = 500
```

The database never observes a partially completed transaction.

### Atomicity Summary

```text
Crash before COMMIT  -> Transaction discarded
Crash after COMMIT   -> Transaction recovered
```

---

# 2. Consistency (Valid State → Valid State)

Consistency guarantees:

> Every committed transaction leaves the database in a valid state according to defined rules and constraints.

## Example: Positive Balance Rule

Business rule:

```text
Account balance cannot be negative.
```

Constraint:

```sql
ALTER TABLE accounts
ADD CONSTRAINT positive_balance
CHECK (balance >= 0);
```

Now suppose a transaction attempts:

```sql
UPDATE accounts
SET balance = -500
WHERE id = 'A';
```

PostgreSQL detects a constraint violation:

```text
ERROR: check constraint violated
```

The update is rejected.

Final state remains:

```text
A = 1000
```

The invalid state never becomes permanent.

---

## Example: Foreign Key Consistency

Tables:

```sql
CREATE TABLE customers (
    id INT PRIMARY KEY
);

CREATE TABLE orders (
    id INT PRIMARY KEY,
    customer_id INT REFERENCES customers(id)
);
```

Attempt to insert an order for a non-existent customer:

```sql
INSERT INTO orders
VALUES (1, 999);
```

Result:

```text
ERROR: foreign key violation
```

Since customer `999` doesn't exist, PostgreSQL rejects the insert.

Consistency is preserved.

---

# How WAL Helps Atomicity

Suppose WAL contains:

```text
TX123:
  A -= 100
  B += 100
```

## Case 1: Crash Before COMMIT

WAL:

```text
TX123 changes
(No COMMIT)
```

During recovery:

```text
Transaction ignored/rolled back
```

Result:

```text
A = 1000
B = 500
```

---

## Case 2: Crash After COMMIT

WAL:

```text
TX123 changes
COMMIT TX123
```

During recovery:

```text
Transaction replayed (redo)
```

Result:

```text
A = 900
B = 600
```

The transaction is never partially applied.

---

# PostgreSQL XID Overflow and MVCC

## The Problem

Transaction IDs (`xmin`, `xmax`) are **32-bit unsigned integers**.

```text
0 .. 4,294,967,295
```

After ~4.2 billion transactions, the next transaction ID wraps around:

```text
4294967295 + 1 = 0
```

If PostgreSQL naively allowed this, visibility checks would break.

Example:

```text
Row xmin = 10
Current xid = 20
```

Clearly transaction `10` is older than `20`.

After wraparound:

```text
Current xid = 5
```

Now transaction `10` appears to be in the future.

This would make MVCC visibility calculations incorrect.

---

## How PostgreSQL Solves It

PostgreSQL treats transaction IDs as a **circular number space**, similar to TCP sequence numbers.

Instead of asking:

```text
Is 10 < 20?
```

it effectively asks:

```text
How far apart are these transaction IDs?
```

As long as the difference is less than roughly **2 billion transactions**, PostgreSQL can determine the correct ordering.

---

## The Real Protection: VACUUM FREEZE

PostgreSQL does not keep ancient transaction IDs forever.

Suppose a row was created by:

```text
xmin = 100
```

Years later:

```text
current xid = 3,000,000,000
```

That old `xmin` becomes dangerous because wraparound is approaching.

At some point, `VACUUM` performs a **freeze operation**:

```text
xmin = FrozenTransactionId
```

This special transaction ID means:

```text
"This row is older than everything."
```

After freezing:

```text
The row is visible to all current and future transactions.
```

PostgreSQL no longer needs to compare the original transaction ID.

---

## Why Autovacuum Is Critical

Autovacuum tracks how old table transaction IDs are becoming.

You can inspect this using:

```sql
SELECT
    relname,
    age(relfrozenxid)
FROM pg_class

---

# Does PostgreSQL Write Reverse Operations?

Usually, **no**.

Some database systems use an UNDO log:

```text
UPDATE A
UNDO A
```

PostgreSQL primarily relies on:

- WAL (for REDO/recovery)
- MVCC (for visibility control)

Instead of undoing every change physically, PostgreSQL stores multiple row versions.
ORDER BY age(relfrozenxid) DESC;
```

As tables approach dangerous transaction
