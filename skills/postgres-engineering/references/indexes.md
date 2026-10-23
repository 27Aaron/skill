# Indexes and Queries

Design indexes from real queries, parameters, and plans, not from column names.

## Index types

| Type | Good for | Notes |
| --- | --- | --- |
| B-tree | Equality, ranges, `IS NULL`, `ORDER BY` | Default; every index adds write and storage cost |
| GIN | Arrays, `jsonb` containment, full-text, `pg_trgm` | Larger and slower to update; check the operator class |
| BRIN | Huge tables whose values correlate with physical order (append-only time series) | Tiny, but useless if the correlation breaks |
| GiST / SP-GiST | Geometry, ranges, nearest-neighbor, exclusion constraints | Capability depends on the operator class |

## Multicolumn B-tree

- Order columns as: equality columns first (often `tenant_id`), then the range or sort column, then a unique tiebreaker.
- Match `ORDER BY` direction so the index can return rows already sorted with `LIMIT`.
- Check for existing indexes the new one makes redundant, or that already cover the query.

```sql
CREATE INDEX orders_pending_created_idx
  ON app.orders (tenant_id, created_at DESC, id DESC)
  WHERE status = 'pending' AND archived_at IS NULL;
```

- **Partial indexes**: the query must provably imply the predicate. A generic prepared plan with `status = $2` can't use the index above; write the literal `status = 'pending'` in the SQL instead.
- **Expression indexes**: the query must use the identical expression, e.g. `lower(email)`.
- **`INCLUDE` columns** enable index-only scans, but only on all-visible pages; check `Heap Fetches` in the plan. Don't build "cover every column" indexes for `SELECT *`; select fewer columns instead.

## Query rewrites

- Look for row explosion from joins, late filtering, and functions or implicit casts on indexed columns (`WHERE date(created_at) = …` → range on `created_at`).
- Don't hide a bad join with `DISTINCT`; use `EXISTS` for semi-joins.
- Verify duplicates, NULLs, ordering, and permissions are unchanged after a rewrite.

## Keyset pagination

Replaces deep `OFFSET`. It needs a unique, non-null, same-direction sort key and an index that matches:

```sql
SELECT id, created_at, total
FROM app.orders
WHERE tenant_id = $1
  AND status = 'pending' AND archived_at IS NULL
  AND (created_at, id) < ($2::timestamptz, $3::bigint)  -- omit on the first page
ORDER BY created_at DESC, id DESC
LIMIT $4;
```

Mixed sort directions need an expanded `OR` condition instead of a row comparison. Keyset gives no snapshot consistency and no random page access.

## Unused indexes

`idx_scan = 0` in `pg_stat_user_indexes` covers only this instance and the time since stats were reset. Before dropping an index, check replicas, periodic jobs, and whether it enforces uniqueness.
