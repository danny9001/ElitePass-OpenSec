# Functional Specification: Database Backups

## Overview
The backup system allows administrators to trigger full and incremental database backups. Backups must run reliably without interference from connection pooler configurations (like PgBouncer).

## Requirements

### Connection String Sanitization
- **Requirement**: The connection string passed to backup CLI utilities (like `pg_dump`) must be clean of any non-standard or connection-pooler parameters.
- **Scenario: Clean Connection String**
  - **Given** the application is configured with a database URL containing parameters like `pgbouncer=true` or `connection_limit=X`
  - **When** a backup is triggered
  - **Then** the backup runner must strip these parameters from the connection URL before passing it to the database utility, ensuring no query parameter validation errors occur.

### PgBouncer Bypassing
- **Requirement**: To prevent connection pool exhaustion and transaction conflicts under transaction pooling, backups should connect directly to the database instance when PgBouncer is configured locally.
- **Scenario: Direct Database Connection**
  - **Given** the database connection points to a local PgBouncer port (e.g., `6432`)
  - **When** a backup is performed
  - **Then** the backup runner must redirect the connection to the direct PostgreSQL port (e.g., `5432`).
