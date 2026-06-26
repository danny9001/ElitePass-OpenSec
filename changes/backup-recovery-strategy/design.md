# Technical Design: ElitePass Backup & Recovery Strategy

This document details the architectural decisions, PostgreSQL command parameters, SQL templates, and Redis recovery configurations to achieve a bulletproof backup and recovery posture.

---

## 1. Pre-Migration PostgreSQL Backup Flow

Database migrations execute against PostgreSQL 16 on `VM00-RESERVAS-v2`. To guarantee a consistent state before schema modifications, a full transactional dump must be taken.

### 1.1 Connection String Sanitization and Port Bypass
All production application connections use PgBouncer on port `6432` running in `transaction` pool mode. Running `pg_dump` through PgBouncer in transaction mode is prone to failure (e.g., query parameter errors, pool starvation, and session-state leaks).

- **Direct Port Redirection**: Redirect connection strings targeting localhost/127.0.0.1 on port `6432` to the raw PostgreSQL engine port `5432`.
- **Parameter Stripping**: Strip parameters such as `?pgbouncer=true` and `&connection_limit=X` from the connection URI to avoid `libpq` connection validation errors.

### 1.2 Transactional State Preservation and Consistency
To ensure the backup is transactionally consistent and represents a single point-in-time snapshot without blocking active writes:
- **Snapshot Isolation**: `pg_dump` natively uses a `REPEATABLE READ` transaction isolation level. This guarantees that all tables dumped represent their state at the exact moment the dump transaction began.
- **De-risking Lock Contention**: The dump should be run using `--serializable-deferrable` (when using serializable transactions) to avoid serialization failures if the database is under heavy concurrent write load.
- **pg_dump Execution Parameters**:
  ```bash
  pg_dump \
    --dbname="postgresql://<user>:<password>@127.0.0.1:5432/<database>?sslmode=disable" \
    --format=custom \
    --compress=9 \
    --no-owner \
    --no-privileges \
    --serializable-deferrable \
    --file="/tmp/pre-migration-snapshot.dump"
  ```
  - `--format=custom` (`-Fc`): Produces a compressed, flexible archive format that supports selective restores and is easily parsed by `pg_restore`.
  - `--no-owner --no-privileges`: Excludes ownership and permission statements, ensuring the backup can be restored onto staging, local environments, or under a different DB user without failure.

### 1.3 Streaming Buffers to Azure Blob Storage
To minimize local disk wear and security exposure:
1. The backup buffer is generated via Node.js `spawn` streaming `stdout` directly from `pg_dump`.
2. The buffer is uploaded to Azure Blob Storage using `@azure/storage-blob` SDK.
3. Path structure: `database-backups/full/jet-club-db-full-YYYY-MM-DD-HH-MM-SS.dump`.
4. The Azure storage account is configured with:
   - **Immutable Storage Policy**: Prevents deletion of backups for 30 days.
   - **Encryption at Rest**: Managed keys via Azure Key Vault.

---

## 2. Database Rollback Procedures

If a Prisma migration or custom DDL script fails mid-execution, we must roll back to a stable schema and data state.

### 2.1 Prisma Migration Resolution
Prisma logs Applied migrations in the `_prisma_migrations` table. If a migration fails, the database is marked as "failed" in this metadata table, blocking subsequent migrations.

1. **Identify the Failed Migration**:
   ```bash
   pnpm prisma migrate status
   ```
2. **Execute Schema Rollback Script**: Apply the SQL rollback script corresponding to the failed migration (see Section 2.2).
3. **Resolve the Status**: Run the following command to mark the failed migration as rolled back, telling Prisma to ignore the failure and allow a clean retry:
   ```bash
   pnpm prisma migrate resolve --rolled-back "20260620120000_failed_migration_name"
   ```

### 2.2 SQL Rollback Templates (DDL and DML)

Every migration folder (e.g., `prisma/migrations/YYYYMMDDHHMMSS_name/`) must include a manual rollback script named `rollback.sql`. Below are standard templates for rollback scenarios.

#### Scenario A: Reverting Schema Alterations (DDL)
```sql
-- rollback.sql for YYYYMMDDHHMMSS_add_fields
BEGIN;

-- 1. Drop constraints and indexes created by the migration
ALTER TABLE "Usuario" DROP CONSTRAINT IF EXISTS "Usuario_licenciaId_fkey";
DROP INDEX IF EXISTS "Usuario_email_idx";

-- 2. Drop columns introduced by the migration
ALTER TABLE "Usuario" DROP COLUMN IF EXISTS "licenciaId";
ALTER TABLE "Usuario" DROP COLUMN IF EXISTS "metadata";

-- 3. Drop tables introduced by the migration
DROP TABLE IF EXISTS "Licencia";

-- 4. Revert custom Enums (if any)
-- Note: PostgreSQL does not support DROP VALUE on Enums easily, so drop the enum type if it was newly created
DROP TYPE IF EXISTS "RolLicencia";

COMMIT;
```

#### Scenario B: Reverting Destructive Schema Changes (Data Preservation)
If a column was renamed or split, data must be restored before dropping the new columns.
```sql
-- rollback.sql for YYYYMMDDHHMMSS_split_name
BEGIN;

-- 1. Restore the original column if it was dropped or renamed
ALTER TABLE "Cliente" ADD COLUMN IF NOT EXISTS "nombreCompleto" VARCHAR(255);

-- 2. Backfill original data from the split columns
UPDATE "Cliente"
SET "nombreCompleto" = TRIM(CONCAT("nombre", ' ', "apellido"))
WHERE "nombreCompleto" IS NULL;

-- 3. Make original column NOT NULL if it was originally constrained
ALTER TABLE "Cliente" ALTER COLUMN "nombreCompleto" SET NOT NULL;

-- 4. Drop the split columns introduced by the migration
ALTER TABLE "Cliente" DROP COLUMN IF EXISTS "nombre";
ALTER TABLE "Cliente" DROP COLUMN IF EXISTS "apellido";

COMMIT;
```

### 2.3 Restoring from Full Pre-Migration Dump
In a catastrophic failure where manual SQL rollbacks are insufficient, restore the database from the pre-migration dump:
```bash
# 1. Terminate all active connections to the database to release locks
psql -h 127.0.0.1 -p 5432 -U postgres -d postgres -c \
  "SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE datname = 'jet_club_db' AND pid <> pg_backend_pid();"

# 2. Perform the clean restore (dropping and recreating tables)
pg_restore \
  -h 127.0.0.1 \
  -p 5432 \
  -U postgres \
  --dbname=jet_club_db \
  --clean \
  --if-exists \
  --no-owner \
  --no-privileges \
  /tmp/pre-migration-snapshot.dump
```

---

## 3. Multi-Tenant Backup Isolation

ElitePass operates under a multi-tenant model. Backups must guarantee tenant-level data security and allow selective recovery without cross-tenant interference.

### 3.1 Logical Backup Isolation
In the POS system (`elitepass-pos`), tenant data is isolated via `empresaId`. The backups service (`backups.service.ts`) executes a logical export of all relevant tables associated with a specific tenant:

1. **Logical Selection**: All records matching the `empresaId` are fetched from tables like `usuario`, `producto`, `venta`, `cajaSesion`, etc., using Prisma.
2. **Metadata Serialization**: Collected data is wrapped in a structured JSON payload detailing schema version, date, and tenant ID:
   ```json
   {
     "version": "1.0",
     "tipo": "EMPRESA",
     "fechaExportacion": "2026-06-20T03:30:00Z",
     "empresaId": "tenant-uuid-123",
     "usuarios": [...],
     "productos": [...]
   }
   ```
3. **Compression & Encryption**: The JSON is compressed with Gzip (`.json.gz`) and streamed to Azure Blob Storage under `backups/empresa/<tenant-slug>/`.

### 3.2 Secure Archiving and Versioning
- **Access Control**: Tenant backups are isolated in separate virtual directories in Azure Blob Storage. Access keys or SAS tokens are generated programmatically with short lifetimes (60 minutes) for downloads.
- **Schema Versioning**: The JSON payload includes a `version` field. When restoring, the recovery worker must verify that the schema version of the backup matches the current database schema.

### 3.3 Selective Tenant Recovery Flow
To restore a single tenant's data without affecting other tenants:
1. **Transaction Isolation**: Wrap the restore in a single database transaction.
2. **Purge Existing Data**: Delete existing records for that `empresaId` in reverse dependency order (child records first, parent records last).
3. **Insert Backup Data**: Insert records from the JSON backup in dependency order (parent records first, child records last) using a generated unique `requestId` to log correlation.
4. **Validation Check**: Verify record counts before and after the transaction commits.

---

## 4. Redis Recovery Guidelines

Redis functions as a transient storage layer on `VM00-RESERVAS-v2` for rate limits (`rl:*`), user sessions, and distributed locks (`lock:*`).

### 4.1 Persistence Configurations (AOF & RDB)
To prevent data loss on Redis crashes, the Redis instance should be configured with hybrid persistence:
- **RDB Snapshotting**: Standard snapshots taken at regular intervals for fast startup.
  ```conf
  save 900 1
  save 300 10
  ```
- **Append-Only File (AOF)**: Writes every incoming command to disk, synced every second. This ensures a maximum of 1 second of lost data.
  ```conf
  appendonly yes
  appendfsync everysec
  no-appendfsync-on-rewrite yes
  auto-aof-rewrite-percentage 100
  auto-aof-rewrite-min-size 64mb
  ```

### 4.2 Application Fallback & Circuit Breaker
If the Redis service goes offline completely:
- The Redis client wrapper (`redis-client.ts`) catches the error event, sets `unavailable = true`, and returns `null`.
- Core services must fall back to in-memory maps (e.g. `memoryStore` in `rate-limit.ts`) or direct PostgreSQL queries for session verification. This keeps the application fully functional during a Redis outage.

### 4.3 Recovery & Warm Start Procedure
When the Redis service is restored:
1. **Service Restart**:
   ```bash
   sudo systemctl restart redis-server
   ```
2. **Orphaned Lock Resolution**:
   - Application-level locks (e.g., ticket seat locks) are set with an explicit TTL (typically 5 to 15 minutes).
   - If Redis crashes, locks are either loaded from the AOF/RDB file or automatically expire when their TTL ends.
   - For immediate clean starts, run the following to flush locks while keeping sessions:
     ```bash
     redis-cli --eval flush-locks.lua
     ```
     *(Where `flush-locks.lua` is a Lua script running `KEYS lock:*` and deleting them).*
3. **Session Re-verification**:
   - User sessions are stored in the PostgreSQL database as the primary source of truth (via better-auth). Redis caches these sessions.
   - If Redis cache is empty, the application reads sessions from PostgreSQL and populates the Redis cache dynamically on the next request. No user sessions are lost.
4. **Cache Warm-up**:
   - Trigger the warm-up endpoints to re-cache active event seating plans and system settings.
