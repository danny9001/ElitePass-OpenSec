# Proposal: Database and Caching Upgrades for ElitePass

## Business Intent
The purpose of this proposal is to design and document database schema extensions, caching layer architectures, and database pool configurations to support high-concurrency operations during peak ticket sales and inventory tracking upgrades in ElitePass POS (`elitepass-pos`) and Reservations (`elitepass-reservas`).

Specifically, this proposal addresses:
1. **FIFO/Batch Inventory Tracking (Lotes)**: Upgrading from simple cumulative stock numbers to structured, expiry-aware, batch-based inventories.
2. **Packaging Mappings**: Enabling automated unit conversions from supplier-purchased packages (e.g. box, keg) to internal inventory units.
3. **Redis Distributed Locks**: Replacing the high-concurrency database-bound `ReservationLock` table with a Redis-backed memory-lock engine to alleviate lock contention on PostgreSQL.
4. **PgBouncer Optimization**: Tuning PgBouncer connection pooling parameters to prevent connection starvation and handle massive concurrent bot/traffic floods.

## Target Architecture & Technologies
- **Databases**: PostgreSQL 16 (relational persistence) & Redis 7.2 (caching and concurrency locking).
- **ORM**: Prisma Client v5+ (for schema management and querying).
- **Middleware**: PgBouncer (database connection pooling in transaction mode).
- **Target Applications**:
  - `elitepass-pos` (inventory, batch tracking, packaging translations).
  - `elitepass-reservas` (high-concurrency ticket and table reservation locking).
