# Security, Permissions, and RLS

Object privileges decide which tables a role can access; RLS decides which rows. Verify both.

## Privileges

- Use separate least-privilege roles for migrations, the app, background jobs, and end users.
- Check `USAGE` on the schema, table and column privileges, sequence privileges, and `EXECUTE` on functions. RLS doesn't restrict columns; protect sensitive columns with column grants.
- Table owners bypass RLS unless `FORCE ROW LEVEL SECURITY` is set. Superusers and `BYPASSRLS` roles always bypass it, so never test RLS as an admin.
- `ALTER DEFAULT PRIVILEGES` affects only future objects created by that role; grant on existing objects separately.

## Policies

- Once RLS is enabled with no applicable policy, everything is denied.
- `USING` filters existing rows (`SELECT`, `UPDATE`, `DELETE`). `WITH CHECK` validates new rows (`INSERT`, `UPDATE`). An `UPDATE` also needs a `SELECT` policy.
- Permissive policies combine with OR, so adding a strict policy doesn't tighten an existing loose one. Restrictive policies combine with AND, and still need at least one permissive policy to allow access.
- Review all existing policies on the table before adding new ones.

## Tenant context (trusted backend only)

Only safe when end users can't run arbitrary SQL, because any session can change a custom GUC. Set the tenant per transaction, with a bound parameter:

```sql
BEGIN;
SELECT set_config('app.tenant_id', $1, true);  -- true = transaction-local
-- queries
COMMIT;
```

```sql
ALTER TABLE app.orders ENABLE ROW LEVEL SECURITY;

CREATE POLICY orders_tenant ON app.orders
  FOR ALL TO app_user
  USING (tenant_id = NULLIF(current_setting('app.tenant_id', true), '')::uuid)
  WITH CHECK (tenant_id = NULLIF(current_setting('app.tenant_id', true), '')::uuid);
```

A missing context becomes `NULL`, which matches no rows. `WITH CHECK` blocks moving a row to another tenant.

## Views and functions

- Views run with the owner's privileges and bypass the caller's RLS. On PG 15+, use `WITH (security_invoker = true)`.
- Use `SECURITY DEFINER` only when elevation is required:
  - Pin `SET search_path = pg_catalog, pg_temp` and schema-qualify every object.
  - Check the caller's identity inside the function.
- New functions grant `EXECUTE` to `PUBLIC`. In the same transaction as `CREATE FUNCTION`, run `REVOKE EXECUTE … FROM PUBLIC` and grant only the intended roles.

## Supabase

Read this section only when the project uses Supabase.

- **Roles**: `anon` and `authenticated` reach tables through the Data API, so tables in exposed schemas need both GRANTs and RLS.
- **Secret keys**: never put `service_role` or the secret key in a client. Testing with them proves nothing, since they bypass RLS.
- **Unauthenticated requests**: `auth.uid()` returns `NULL`, and a policy that evaluates to `NULL` denies.
- **Tenants**: a user ID is not a tenant ID; check membership through a join table.
- **JWT metadata**: never authorize on `user_metadata`, which users can edit. Even `app_metadata` claims stay stale until the token refreshes.
- **Policy performance**: wrap row-independent calls as `(SELECT auth.uid())`, so they're evaluated once per statement instead of once per row.
- **Private helpers**: put `SECURITY DEFINER` helpers in a schema the API doesn't expose.
- **Connections**: use the direct or session-mode connection for migrations; transaction mode doesn't keep session state. Check current Supabase docs rather than relying on memory.

## Verify

- Test as the real role, through the real connection path.
- Cover same tenant, other tenant, missing context, changing `tenant_id`, and the joins, views, and functions actually used.
- Run two tenants back to back on one pooled connection to confirm no context leaks.
- Tell apart three outcomes: 0 rows (filtered), a `WITH CHECK` violation, and a permission error.
