# Proposal: Fix PgBouncer backup error in ElitePass Reservas

## Problem Description
In the reservation system, taking a backup from **Configuración > Backups** fails with the following error:
```
pg_dump: error: invalid URI query parameter: "pgbouncer"
```

This error happens because the backup utility (`pg_dump`) is invoked with the full application connection string defined in `DATABASE_URL` (e.g., `postgresql://admin:***@localhost:6432/jet_club_db?pgbouncer=true&connection_limit=5`). `pg_dump` uses `libpq` for database connections, which does not recognize or support query parameters like `pgbouncer` or `connection_limit`.

## Business Intent
Ensure that system administrators can successfully take on-demand and automated database backups from the administration dashboard without encountering PgBouncer parameter validation errors.
