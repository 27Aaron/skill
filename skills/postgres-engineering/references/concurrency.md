# Concurrency, Transactions, and Locks

## Transactions

- Keep transactions short. Never hold one open across network calls or user input.
- Lock rows in a consistent order to avoid deadlocks.
- Choose the isolation level for the invariant. Under `READ COMMITTED`, each statement sees a new snapshot.
- **Optimistic locking**: `UPDATE … SET version = version + 1 WHERE id = $1 AND version = $2`. If 0 rows are updated, tell apart not found, forbidden, and stale version.
- **Single-row invariants**: use a conditional atomic update, e.g. `UPDATE stock SET qty = qty - $2 WHERE id = $1 AND qty >= $2`.
- **Insert-or-update**: use `INSERT … ON CONFLICT` backed by a unique index, never select-then-insert.

## Retries

On SQLSTATE `40001` (serialization failure) or `40P01` (deadlock), roll back and retry the whole transaction — including the logic that built it — with backoff and an overall deadline. Don't retry unique violations blindly. If the commit outcome is unknown (connection lost), check an idempotency key or the business state before redoing external effects.

## Diagnosing lock waits

```sql
SELECT pid, state, wait_event_type, wait_event,
       now() - xact_start AS xact_age,
       pg_blocking_pids(pid) AS blocked_by,
       left(query, 80) AS query
FROM pg_stat_activity
WHERE datname = current_database() AND wait_event_type = 'Lock'
ORDER BY xact_start;
```

- Follow `blocked_by` to the root blocker. It's often `idle in transaction`, holding locks from an earlier statement.
- `pg_cancel_backend` cancels only the current statement; the transaction keeps its locks.
- Terminating a session is a last resort and needs approval.

## Job queue

```sql
WITH next AS (
  SELECT id FROM app.jobs
  WHERE status = 'pending' AND run_at <= now()
  ORDER BY run_at, id
  FOR UPDATE SKIP LOCKED
  LIMIT 1
)
UPDATE app.jobs j
SET status = 'running', claimed_at = now(), lease_token = gen_random_uuid()
FROM next WHERE j.id = next.id
RETURNING j.*;
```

- Commit the claim, then do the work outside that transaction.
- Back it with an index on `(status, run_at, id)`.
- `SKIP LOCKED` is not strict FIFO and not exactly-once. For lease expiry, match `lease_token` on heartbeat and completion, so a stale worker can't overwrite a newer claim.
- Make external effects idempotent, and cap retries with a dead-letter state.

## Advisory locks

- Use advisory locks for app-level mutual exclusion, with a documented key space.
- Behind a transaction pooler, use only `pg_advisory_xact_lock`. Session locks survive rollback, and the next request may get a different connection.
- They don't replace constraints.
