# Monitoring and Maintenance

Base thresholds on table size, write rate, and replication setup, not fixed numbers.

## What to watch

- **Queries**: total and mean time and calls (`pg_stat_statements`); p95/p99 from app tracing.
- **Resources**: CPU, I/O, cache hit ratio, temp files, WAL volume, replication lag.
- **Concurrency**: active connections, lock waits, long transactions, deadlocks, serialization failures.
- **Maintenance**: last (auto)vacuum and (auto)analyze, dead tuples, `age(relfrozenxid)`, table and index growth.

## VACUUM and ANALYZE

- Keep autovacuum on. For hot, large tables, tune per-table `autovacuum_vacuum_scale_factor` and `autovacuum_vacuum_threshold` rather than globally.
- `n_dead_tup` is an estimate, not bytes of bloat. Use size trends (or `pgstattuple`) to measure bloat.
- `ANALYZE` refreshes planner stats; run it after bulk loads or big distribution changes. It doesn't reclaim space.
- Plain `VACUUM` makes space reusable within the table but rarely shrinks the file.
- `VACUUM FULL` rewrites the table under `ACCESS EXCLUSIVE` and needs extra disk — not an online fix. Prefer `pg_repack` when bloat must be removed online.
- **When vacuum can't keep up**, look for what holds back the xmin horizon: long transactions, abandoned replication slots, `hot_standby_feedback`, and prepared transactions.
- **When `age(datfrozenxid)` approaches `autovacuum_freeze_max_age`**, treat wraparound as urgent. Never disable autovacuum.
- Do big deletes and updates in batches, watching WAL and replication lag.

## Alerts

- An alert names the object, the observed value, the baseline, a likely cause, and a safe next step.
- After maintenance, confirm the metrics actually improved. A faster query that brings worse locking or WAL is not a win.
