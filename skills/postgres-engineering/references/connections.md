# Connections and Pooling

More connections is not more capacity. Size the pools, then verify with metrics.

## Budget

- **Direct connections**: add up replicas × processes × pool size across every service. Include extra replicas during rolling deploys, cron jobs, and admin tools, and keep headroom under `max_connections`.
- **With a pooler**: client connections and server connections are separate budgets. PgBouncer's `default_pool_size` applies per database/user pair and per PgBouncer instance.
- **Serverless or bursty workloads**: limit concurrency upstream instead of raising `max_connections`.
- `idle in transaction` sessions are worse than idle ones, because they hold locks and block vacuum.

## Transaction pooling caveats

The server connection changes between transactions, so these don't work across transactions:

- Session `SET`: use `SET LOCAL` or `set_config(…, true)` inside the transaction.
- `LISTEN`/`NOTIFY`, session advisory locks, temp tables that outlive a transaction, and SQL-level `PREPARE`.
- Protocol-level prepared statements work in PgBouncer 1.21+ only with `max_prepared_statements > 0`; other poolers differ.

Use a session-mode or direct connection for migrations and anything that needs session state.

## Timeouts

- Set `statement_timeout`, `lock_timeout`, and `idle_in_transaction_session_timeout` per role or session, at values that fit the workload. Avoid `ALTER SYSTEM` on managed platforms.
- Keep `lock_timeout` below `statement_timeout`.
- Set separate limits for pool acquire, lock wait, statement, and upstream calls.
- A client timeout doesn't stop the server. Roll back or discard a failed connection before returning it to the pool.
- `work_mem` applies per sort or hash node, per worker — not per connection. Estimate the peak from concurrency before raising it.

## Diagnose

Tell apart pool exhaustion, slow queries, lock waits, and network queuing, using `pg_stat_activity`, pooler stats, and DB CPU and I/O. Raising the pool size makes the last three worse.
