# Proposal: ElitePass Backup & Recovery Strategy

## Context & Business Intent
ElitePass relies on high availability and zero-data-loss guarantees for its database operations (user authentication, ticket reservations, payment logs, and POS transactional records). Database migrations are critical inflection points. An unmitigated migration failure or cache crash could lead to extended downtime, data inconsistencies, or double-spending/lost orders in our multi-tenant environments.

To guarantee operational resilience, we are establishing a comprehensive, spec-driven Backup & Recovery Strategy.

## Objectives
1. **Pre-Migration Backups**: Establish a reliable, automated, and tested procedure for creating full transactional backups directly from PostgreSQL before applying any schema or data migrations.
2. **Deterministic Rollbacks**: Detail steps to safely revert migrations (using both automated Prisma flows and custom SQL rollback templates) if a migration fails in production.
3. **Multi-Tenant Backup Isolation**: Ensure backups are securely archived, versioned, and that tenant-level logical data is isolatable for selective restoration.
4. **Redis Cache & State Restoration**: Define disaster recovery mechanisms for transient keys (locks, rate limits, sessions) stored in Redis in case of crashes or connection dropouts.

## Scope
- Applies to all core database instances on `VM00-RESERVAS-v2` (PostgreSQL 16).
- Establishes guidelines for the application of Prisma migrations and emergency rollbacks.
- Outlines multi-tenant isolation for backups in `elitepass-pos` and `elitepass-reservas`.
- Provides explicit recovery steps for Redis transient states.

## Constraints
- **PgBouncer Bypass**: Database dumps must run on the direct port `5432` rather than the connection pooler port `6432` to prevent connection exhaustion and transactional pooling incompatibilities.
- **Azure Blob Storage**: Backup streams must upload directly to Azure Blob Storage using secure SAS tokens without leaving unencrypted temporary files on disk.
- **Spec-Driven**: The entire strategy must be fully documented, reproducible, and tracked via OpenSpec.
