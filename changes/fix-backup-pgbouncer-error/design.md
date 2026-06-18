# Technical Design: Clean Connection Strings and Direct Connection for Backups

## Root Cause Analysis
- The application's `DATABASE_URL` contains `?pgbouncer=true&connection_limit=5`.
- When `pg_dump` runs, these query parameters are passed in the connection string argument `--dbname`.
- `libpq` throws a fatal error because they are invalid query parameters.
- Furthermore, PgBouncer on VM00 runs on port `6432` in `transaction` pool mode. Running `pg_dump` through a transaction-pooled connection can lead to connection starvation or transaction errors, as `pg_dump` relies on long-running, session-wide repeatable-read transactions.

## Technical Decisions
1. **Clean Database Connection String**: Implement a helper function `getCleanDatabaseUrl()` in `src/lib/backups/database-backup.ts` to parse the `DATABASE_URL` and strip parameters unsupported by `pg_dump` (like `pgbouncer` and `connection_limit`).
2. **Direct Connection Redirect**: If the database connection host is `localhost` or `127.0.0.1` and the port is `6432` (PgBouncer port), automatically rewrite the port to `5432` (PostgreSQL direct port). This bypasses PgBouncer entirely, avoiding pool starvation and transaction mode issues.
3. **Pass Clean URL to pg_dump**: Modify `createFullBackupBuffer` to pass this cleaned/redirected connection string instead of the raw `DATABASE_URL`.
