# EXPLAIN and Performance Diagnosis

Tie every conclusion to the real SQL, parameters, data size, and version. Plan cost is an estimate, not time.

## Capture

1. Record the SQL, representative parameters, call rate, latency target, row count, role, RLS, and whether prepared statements are used.
2. Start with plain `EXPLAIN` (doesn't execute).
3. For real timings, in a safe environment:

```sql
EXPLAIN (ANALYZE, BUFFERS, SETTINGS)
SELECT id, total
FROM app.orders
WHERE tenant_id = $1 AND status = $2
ORDER BY created_at DESC, id DESC
LIMIT 50;
```

- In `psql`, test parameterized SQL with `PREPARE q(uuid, text) AS …; EXPLAIN ANALYZE EXECUTE q(…);`. A literal-value plan can differ from the generic plan the app actually uses.
- Wrap write statements in `BEGIN; … ROLLBACK;`, but remember that side effects such as sequences and external calls persist.

## What to read

- **Estimated vs actual rows**: a large gap points to stale stats, skew, or correlated columns. Fix with `ANALYZE`, a higher per-column statistics target, or `CREATE STATISTICS`.
- **`loops`**: actual rows and time are per loop. A parent node's time includes its children's.
- **Scan choice**: a `Seq Scan` is right when most rows are needed. Many `Rows Removed by Filter` on a selective predicate suggests an index.
- **Sort / Hash**: `external merge` or `Batches > 1` means spilling to disk; consider a matching index or a session-level `work_mem` bump.
- **Buffers**: `shared read` may still come from the OS cache. `temp read/written` means spills.
- Planning time, parallel workers, and join order.

Use `enable_seqscan = off` and similar settings only to diagnose, never as a fix.

## Find what to optimize

Rank queries in `pg_stat_statements` by `total_exec_time`, then `mean_exec_time` and `calls`. It needs `shared_preload_libraries` (requires a restart) plus `CREATE EXTENSION`; check before assuming it's available. It reports averages, not p95/p99; take percentiles from app tracing or logs.

## Measure

- Compare before and after with the same data, params, and cache state. Run several times; look at p95/p99, throughput, and lock waits, not a single run.
- `EXPLAIN ANALYZE` discards results, so check result equivalence separately. End-to-end latency also includes pool wait, network, and client time.
- Save the plan, version, and data size as evidence.
