# Transactions — ACID, Isolation Levels, Concurrency Control

---

## 1. What is a Transaction?

A transaction is a sequence of DB operations executed as a **single logical unit**. Either all succeed (commit) or all fail (rollback).

```sql
BEGIN TRANSACTION;
    UPDATE accounts SET balance = balance - 500 WHERE id = 1;  -- debit
    UPDATE accounts SET balance = balance + 500 WHERE id = 2;  -- credit
COMMIT;
-- If any step fails, ROLLBACK ensures neither update persists
```

---

## 2. ACID Properties

| Property | Meaning | Mechanism |
|----------|---------|-----------|
| **Atomicity** | All or nothing — partial updates never visible | Undo log / rollback |
| **Consistency** | Transaction moves DB from one valid state to another | Constraints, triggers, application logic |
| **Isolation** | Concurrent transactions appear to run serially | Locks, MVCC |
| **Durability** | Committed data survives crashes | Write-Ahead Log (WAL), fsync |

---

## 3. Concurrency Problems

| Problem | Description | Example |
|---------|-------------|---------|
| **Dirty Read** | Read uncommitted data from another transaction | T1 writes, T2 reads before T1 commits, T1 rolls back |
| **Non-repeatable Read** | Same row read twice in one transaction gives different values | T2 updates a row between T1's two reads |
| **Phantom Read** | Same query returns different set of rows | T2 inserts rows between T1's two range queries |
| **Lost Update** | Two transactions both read and write, one's update is lost | T1 and T2 both read balance=100, both add 50, final=150 not 200 |

---

## 4. Isolation Levels

Defined by SQL standard. Higher isolation = fewer anomalies = lower concurrency.

| Level | Dirty Read | Non-repeatable | Phantom | Use Case |
|-------|-----------|----------------|---------|----------|
| **READ UNCOMMITTED** | ✅ possible | ✅ possible | ✅ possible | Analytics (rarely used) |
| **READ COMMITTED** | ❌ prevented | ✅ possible | ✅ possible | Default in PostgreSQL, Oracle |
| **REPEATABLE READ** | ❌ | ❌ prevented | ✅ possible | Default in MySQL InnoDB |
| **SERIALIZABLE** | ❌ | ❌ | ❌ prevented | Financial critical |

```sql
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
BEGIN;
-- ...
COMMIT;
```

---

## 5. Two-Phase Locking (2PL)

**Protocol:** Transactions acquire all locks before releasing any.

**Phase 1 — Growing:** Acquire locks (shared/exclusive), never release.
**Phase 2 — Shrinking:** Release locks, never acquire.

```
Transaction T:
    lock(A, shared)      ← growing phase
    read(A)
    lock(B, exclusive)
    write(B)
    unlock(A)            ← shrinking phase
    unlock(B)
```

**Guarantees:** Conflict serializability (no dirty reads, no lost updates).
**Problem:** Can cause deadlocks (cycle of transactions waiting for each other's locks).

**Strict 2PL:** Hold all exclusive locks until commit/abort. Prevents cascading aborts.

---

## 6. MVCC — Multi-Version Concurrency Control

Used by PostgreSQL, MySQL InnoDB, Oracle. Readers don't block writers; writers don't block readers.

**Idea:** Instead of overwriting a row, create a new version. Each transaction sees a consistent snapshot of the DB from when it started.

```
Row versions:
  id=1, balance=100, txn_id=5,  valid_until=10
  id=1, balance=150, txn_id=10, valid_until=∞   ← current

Transaction started at txn_id=7 reads version with txn_id=5 (100)
Transaction started at txn_id=11 reads version with txn_id=10 (150)
```

**Garbage collection:** Old versions removed by vacuum/autovacuum.

---

## 7. Write-Ahead Log (WAL)

**Rule:** Log the change to disk before applying it to the actual data page.

```
Recovery after crash:
1. Scan WAL from last checkpoint
2. REDO all committed transactions
3. UNDO all uncommitted transactions
```

This guarantees **durability** (committed transactions survive crashes) and **atomicity** (incomplete transactions are rolled back).

---

## 8. Savepoints

```sql
BEGIN;
INSERT INTO orders VALUES (1, 'Alice', 100);
SAVEPOINT sp1;
INSERT INTO orders VALUES (2, 'Bob', 200);
-- Something fails
ROLLBACK TO sp1;  -- undo Bob's insert, keep Alice's
COMMIT;           -- commit Alice's insert
```

---

## 9. Distributed Transactions — 2PC

**Two-Phase Commit:** Coordinates commit across multiple DB nodes.

```
Phase 1 — Prepare:
    Coordinator → all participants: "Can you commit?"
    Each participant logs "ready" and replies YES/NO

Phase 2 — Commit/Abort:
    If all YES: Coordinator → "COMMIT"
    If any NO:  Coordinator → "ABORT"
```

**Problem:** Coordinator can fail in phase 2 → participants are stuck (blocking protocol). Modern systems use Paxos/Raft for non-blocking consensus.

---

## 10. Quick Reference

| Question | Answer |
|----------|--------|
| What is a dirty read? | Reading uncommitted data that may be rolled back |
| What does MVCC stand for? | Multi-Version Concurrency Control |
| How does Postgres prevent dirty reads? | Read Committed isolation + MVCC (see committed snapshot) |
| What is a deadlock in DB? | Two transactions each waiting for a lock the other holds |
| WAL vs undo log? | WAL = redo after crash; undo log = rollback uncommitted |
| BASE vs ACID? | BASE = Basically Available, Soft state, Eventually consistent — used in NoSQL |
