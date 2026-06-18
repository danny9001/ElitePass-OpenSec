# Design: Database Backup Error Fix

## Root Cause Analysis
During incremental backups, the query `SELECT * FROM <tableName> ...` returns raw database rows. Some tables (e.g. `Passkey`) contain columns of type `BigInt` (e.g. `counter` in Prisma/Postgres). The default `JSON.stringify` in `database-backup.ts:197` does not support `BigInt` type serialization and throws:
`TypeError: Do not know how to serialize a BigInt`.

## Technical Decisions
1. **Custom JSON Replacer**: Introduce a custom JSON replacer in `JSON.stringify()` inside `createIncrementalBackupBuffer` to convert `BigInt` values to strings:
   ```typescript
   function jsonSafeReplacer(key: string, value: unknown) {
     if (typeof value === "bigint") {
       return value.toString();
     }
     return value;
   }
   ```
   This prevents serialization failure.

2. **Verify Middleware / Token Security & CSRF Bypass**:
   The backup route `/api/backups/run` validates the secret via a `Bearer` token passed in the `Authorization` header, matched against `process.env.BACKUP_CRON_SECRET`.
   However, `middleware.ts`'s `validateCsrf` rejects all `POST` requests without valid `Origin` or `Referer` headers. Since the cron job uses `curl -X POST` locally, it fails with a 403 Forbidden.
   To fix this:
   - We will modify `validateCsrf` in `src/middleware.ts` to bypass CSRF checks if the request path starts with `/api/backups/`.
   - This keeps the route secure because `/api/backups/run` itself validates the `BACKUP_CRON_SECRET` bearer token, while allowing automated curl triggers from localhost/cron.

3. **Verify Cron Execution**:
   Verify that local curl request with authorization header succeeds.

