# Safe Migrations

First check table size, write load, the migration tool's transaction mode, and how the app deploys. Small or empty tables can use plain transactional DDL; stage changes only when old and new app versions must coexist.

## Expand / contract

1. **Expand**: add nullable columns, tables, and indexes; drop nothing.
2. **Write and backfill**: deploy code that writes the new shape, then backfill in batches.
3. **Switch**: move reads to the new shape and verify consistency.
4. **Contract**: once no old app version is running, drop the old columns and code.

**Backfills**:
- Batch by primary key range (not `OFFSET`), committing each batch, throttled.
- Make reruns skip finished rows (`WHERE new_col IS NULL`), and never overwrite newer app writes.
- Watch replication lag, WAL, and locks; pause on thresholds.

## Locks and DDL

- Most `ALTER TABLE` forms take `ACCESS EXCLUSIVE`. Even a fast one queues behind long transactions and then blocks everything behind it. Always set:
  ```sql
  SET lock_timeout = '5s';
  ```
  On timeout, find the blocker and retry with backoff; don't retry forever.
- Adding a column with a constant default is metadata-only (PG 11+). A volatile default (`now()` is fine; `random()` and `gen_random_uuid()` aren't) or `ALTER COLUMN TYPE` rewrites the table.

## Indexes

- Use `CREATE INDEX CONCURRENTLY` on live tables. It **can't run inside a transaction block**, so the migration step must be non-transactional. It still waits for existing transactions.
- A failed build leaves an `INVALID` index. Check before retrying:
  ```sql
  SELECT indexrelid::regclass, indisvalid, pg_get_indexdef(indexrelid)
  FROM pg_index WHERE indrelid = 'app.orders'::regclass;
  ```
  Drop the invalid index with `DROP INDEX CONCURRENTLY`, then rebuild. `IF NOT EXISTS` would skip it silently.

## Constraints without long locks

`CHECK` and foreign keys: add `NOT VALID`, which checks new writes only, then `VALIDATE CONSTRAINT` in a separate transaction under a weaker lock.

Making a column `NOT NULL` on a large table (after the backfill, with no new NULLs being written):

```sql
ALTER TABLE app.orders ADD CONSTRAINT orders_tenant_id_nn
  CHECK (tenant_id IS NOT NULL) NOT VALID;
ALTER TABLE app.orders VALIDATE CONSTRAINT orders_tenant_id_nn;
ALTER TABLE app.orders ALTER COLUMN tenant_id SET NOT NULL;  -- PG 12+: no scan
ALTER TABLE app.orders DROP CONSTRAINT orders_tenant_id_nn;
```

Run each statement in its own transaction.

Unique constraints: first `CREATE UNIQUE INDEX CONCURRENTLY`, then `ALTER TABLE … ADD CONSTRAINT … UNIQUE USING INDEX …`. Partial and expression indexes can't be attached this way.

## Failure and rollback

- If an object exists with a different definition, stop and report; don't paper over it.
- Before dropping or renaming, check apps, views, functions, triggers, and reports that reference the object.
- A failed non-transactional migration may be partly applied; inspect the catalog to see which steps ran.
- Down migrations can't restore dropped data. Choose roll back, roll forward, or restore from backup, and state the cost.

For risky production migrations, document the expected locks, duration, WAL and replication impact, pause conditions, and recovery path. Rehearse on production-like data when possible.
