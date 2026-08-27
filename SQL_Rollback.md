# Interview Answer

> PostgreSQL writes WAL records as changes occur. If a transaction is rolled back, PostgreSQL writes an ABORT record to WAL. Unlike databases that rely heavily on UNDO logs, PostgreSQL uses MVCC, where updates create new row versions instead of overwriting existing rows. When a transaction aborts, its row versions simply become invisible and are ignored during normal execution and crash recovery. If PostgreSQL finds WAL records without a corresponding COMMIT during recovery, those changes are treated as aborted and are not applied, ensuring atomicity.

# PostgreSQL WAL and Rollback

A common question is:

> If PostgreSQL writes transaction changes to WAL first, and later the transaction fails and is rolled back, what happens to the WAL? Does PostgreSQL write another WAL record?

The answer is **yes**. PostgreSQL writes an **ABORT (rollback) record** to WAL.

---

# Scenario

Consider the following transaction:

```sql
BEGIN;

UPDATE accounts
SET balance = 900
WHERE id = 'A';

UPDATE accounts
SET balance = 600
WHERE id = 'B';

ROLLBACK;
```

---

# What Gets Written to WAL?

As PostgreSQL executes the updates, it generates WAL entries describing the changes.

Conceptually:

```text
TX123:
  UPDATE A
  UPDATE B
```

When the transaction is rolled back:

```sql
ROLLBACK;
```

PostgreSQL writes an abort marker:

```text
TX123:
  UPDATE A
  UPDATE B
  ABORT
```

So yes, PostgreSQL records the rollback decision in WAL.

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

---

# MVCC Example

Original row:

```text
Account A = 1000
```

Transaction TX123 updates it:

```text
Account A = 900
```

Internally PostgreSQL creates:

```text
Old Version: balance = 1000
New Version: balance = 900 (created by TX123)
```

The old version is not immediately destroyed.

---

# What Happens on Rollback?

When TX123 aborts:

```text
ABORT TX123
```

The new row version becomes invalid/invisible.

Other transactions continue seeing:

```text
balance = 1000
```

The aborted version is ignored.

No expensive physical undo operation is required.

---

# What If the Server Crashes Before Rollback?

Consider:

```sql
BEGIN;

UPDATE A;
UPDATE B;
```

WAL contains:

```text
UPDATE A
UPDATE B
```

Before COMMIT is written, the server crashes.

---

## Recovery Process

After restart PostgreSQL examines WAL:

```text
TX123:
  UPDATE A
  UPDATE B

(No COMMIT record)
```

Since there is no COMMIT:

```text
Transaction is treated as aborted
```

The row versions created by TX123 are ignored.

Final result:

```text
A = original value
B = original value
```

---

# Why This Works

Traditional approach:

```text
Modify existing row
↓
Need UNDO log to restore old value
```

PostgreSQL approach:

```text
Keep old row version
Create new row version
Track transaction status
```

If transaction commits:

```text
New version becomes visible
```

If transaction aborts:

```text
New version becomes invisible
```

This is one of the major benefits of MVCC.

---

# Mental Model

Think of a transaction as creating a draft.

Original:

```text
Balance = 1000
```

Transaction draft:

```text
Balance = 900
```

### Commit

```text
Draft approved
```

WAL:

```text
COMMIT TX123
```

Result:

```text
Balance = 900
```

### Rollback

```text
Draft discarded
```

WAL:

```text
ABORT TX123
```

Result:

```text
Balance = 1000
```

The original version was never overwritten.

---

# Interview Answer

> PostgreSQL writes WAL records as changes occur. If a transaction is rolled back, PostgreSQL writes an ABORT record to WAL. Unlike databases that rely heavily on UNDO logs, PostgreSQL uses MVCC, where updates create new row versions instead of overwriting existing rows. When a transaction aborts, its row versions simply become invisible and are ignored during normal execution and crash recovery. If PostgreSQL finds WAL records without a corresponding COMMIT during recovery, those changes are treated as aborted and are not applied, ensuring atomicity.
