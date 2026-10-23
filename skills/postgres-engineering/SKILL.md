---
name: postgres-engineering
description: Design, implement, or review PostgreSQL schemas, SQL, indexes, migrations, RLS, and transactions; diagnose slow queries, lock waits, connection exhaustion, and vacuum issues. Covers Supabase's database layer. Not for other databases or app-only design.
metadata:
  author: Aaron
  version: "1.0.0"
---

# PostgreSQL Engineering

Follow the project's PostgreSQL major version, migration tool, naming conventions, and role model.

## Context

Gather only what changes the decision:

| Task | Need |
| --- | --- |
| Schema / SQL | Business invariants, related tables, constraints, indexes, real call patterns |
| Performance | SQL, representative params, data size and distribution, plan, call rate, latency target |
| Migration | Major version, tool's transaction mode, table size, write load, deploy compatibility window |
| Permissions / RLS | Actual login and executing roles, GRANTs, all policies, where tenant identity comes from |
| Connections / maintenance | Pooler and mode, app replica count, connection, lock, and vacuum stats |

A missing plan doesn't block fixing a provable SQL or permission bug; label performance advice as unverified instead.

## References

Read only what the task needs:

| Topic | File |
| --- | --- |
| Types, constraints, foreign keys, multi-tenant keys | [schema.md](references/schema.md) |
| Index design, query rewrites, keyset pagination | [indexes.md](references/indexes.md) |
| `EXPLAIN`, `pg_stat_statements`, benchmarking | [explain.md](references/explain.md) |
| Online DDL, backfills, constraint rollout | [migrations.md](references/migrations.md) |
| GRANTs, RLS, views, `SECURITY DEFINER`, Supabase | [security.md](references/security.md) |
| Transactions, retries, locks, job queues | [concurrency.md](references/concurrency.md) |
| Connection budgets, pooler compatibility, timeouts | [connections.md](references/connections.md) |
| Statistics, VACUUM, bloat, monitoring | [maintenance.md](references/maintenance.md) |

## Working rules

- Edit the project's actual SQL, migrations, or calling code, and run the relevant checks.
- Correctness first: result set, ordering, NULL semantics, constraints, and permissions. Then performance.
- Prove performance changes with comparable data, params, and role; a lower cost estimate or an `Index Scan` alone isn't proof.
- Test RLS as the real role, on both allow and deny paths. Test concurrency with interleaved sessions.
- Bind values as `$1` parameters. Never use a value parameter for an identifier; use an allowlist plus `format('%I', …)`.
- If no database is available, do static checks and say what wasn't run.

## Safety

- Writing a migration is not running it; don't execute against production unless asked.
- `EXPLAIN ANALYZE` executes the statement, including writes inside functions; rollback doesn't undo sequence use or external effects.
- Don't widen grants, disable RLS, run `ALTER SYSTEM`, or use `CASCADE` to silence dependency errors.
- `IF NOT EXISTS` doesn't verify that the existing object has the expected definition; check the catalog.
- Strip credentials and user data from examples and evidence.

## Output

- **Implementation**: what changed, why, and how it was verified. For migrations, add execution order, lock impact, and recovery steps.
- **Review**: findings by impact, each with location, trigger, evidence, and minimal fix. Keep confirmed issues separate from hypotheses.
- Keep small fixes short; no rollback plan for tasks without a migration.
