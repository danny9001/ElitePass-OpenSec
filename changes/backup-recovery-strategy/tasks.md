# Tasks: ElitePass Backup & Recovery Strategy

## Phase 1: Pre-Migration Backup Setup & Validation
- [ ] Implement connection string cleaning in `database-backup.ts`:
  - [ ] Strip `pgbouncer` and `connection_limit` parameters.
  - [ ] Rewrite localhost port from `6432` (PgBouncer) to `5432` (PostgreSQL direct).
- [x] Configure `pg_dump` parameters for transactional consistency:
  - [x] Use custom format (`--format=custom`) and high compression.
  - [x] Add `--no-owner --no-privileges` to output.
  - [x] Add `--serializable-deferrable` for concurrency safety.
- [ ] Configure streaming upload to Azure Blob Storage:
  - [ ] Stream stdout bytes directly to block blob client.
  - [ ] Enforce naming convention: `database-backups/full/jet-club-db-full-YYYY-MM-DD-HH-MM-SS.dump`.
- [x] Verify execution by running `test-backup-error.ts` and confirming the dump file is created successfully in Azure Blob Storage.

## Phase 2: Rollback Procedures Verification
- [ ] Standardize the creation of `rollback.sql` for all new database migrations:
  - [ ] Include SQL DDL statements to drop columns/tables, indexes, and custom types.
  - [ ] Wrap all rollback operations in a single `BEGIN/COMMIT` transaction block.
- [ ] Document the use of Prisma CLI for migration state management:
  - [ ] Verify `pnpm prisma migrate status` reports failed states.
  - [ ] Validate `pnpm prisma migrate resolve --rolled-back <migration_name>` command resets the migration state in `_prisma_migrations`.
- [ ] Validate standard `pg_restore` command with `--clean` and `--if-exists` flags against a test database environment.

## Phase 3: Multi-Tenant Backup Isolation
- [ ] Verify the logical export pipeline:
  - [ ] Verify `crearBackupEmpresa` in `backups.service.ts` successfully fetches all records matching a specific `empresaId`.
  - [ ] Verify JSON structure contains a `version` field and is compressed with Gzip before upload.
- [ ] Define the tenant restore script flow:
  - [ ] Enforce schema version validation before executing restore.
  - [ ] Run deletion of existing tenant data in reverse-dependency order within an interactive transaction.
  - [ ] Re-insert backup records inside the same transaction block.

## Phase 4: Redis State Recovery & Resilience
- [ ] Verify Redis persistence configuration:
  - [ ] Enable AOF (`appendonly yes` and `appendfsync everysec`).
  - [ ] Enable RDB snapshots (`save 900 1` and `save 300 10`).
- [ ] Verify application-level circuit breaker:
  - [ ] Simulate Redis crash by stopping Redis (`sudo systemctl stop redis-server`).
  - [ ] Verify that request flow fallback to local memory cache (`checkMemory` in `rate-limit.ts`) works without throwing HTTP 500 errors.
- [ ] Verify cache rebuilds when Redis is restarted:
  - [ ] Restart Redis and check that DB-backed sessions are cached dynamically on demand.
