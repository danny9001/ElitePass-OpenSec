# Tasks: System Upgrades Integration Plan

## 1. Analysis and Schema Design
- [ ] Review existing Prisma schemas in `elitepass-pos` and `elitepass-reservas` to ensure no conflict with new tables.
- [ ] Create the database migration script for `LoteProducto`, `KardexLote`, and `SupplierProduct` tables.
- [ ] Establish connection strings and authentication settings for Redis 7.2 in the `.env` configs.

## 2. Redis-Based Concurrency Lock Engine
- [ ] Define the `ILockManager` interface.
- [ ] Implement `RedisLockManager` (using `ioredis` and a Lua-script release mechanism).
- [ ] Implement `PostgresLockManager` (using the legacy `ReservationLock` DB schema).
- [ ] Create the `ReservationLockService` orchestrator that toggles engines via the `USE_REDIS_LOCKS` flag.
- [ ] Write integration tests for the lock engine to verify tenant key namespaces.
- [ ] Deploy Phase 1 (Dual-Write & Compare).
- [ ] Deploy Phase 2 (Redis Active, SQL Fallback).
- [ ] Deploy Phase 3 (Redis-Only, SQL Async archiving).
- [ ] Deploy Phase 4 (Drop `ReservationLock` table).

## 3. Batch/FIFO Inventory System
- [ ] Define the `IInventoryConsumptionStrategy` interface.
- [ ] Implement `FifoStrategy` and `FefoStrategy` in the domain layer.
- [ ] Implement `LoteRepository` in the infrastructure layer.
- [ ] Update `PurchaseService` to write to both aggregate inventory and batch tables (Phase 2).
- [ ] Implement the async reconciliation job `KardexReconciliation` (Phase 2).
- [ ] Execute the `LOT-LEGACY` historical stock migration seed script (Phase 3).
- [ ] Switch the POS inventory read path to use batch-first queries (Phase 4).
- [ ] Deprecate the legacy `stock` column in `Producto` (Phase 5).

## 4. Turnstile Bot Verification Wrapper
- [ ] Deploy the verification logic API route `verifyTurnstileToken` (disabled / logging-only mode).
- [ ] Integrate the Turnstile component widget on frontend reservation forms.
- [ ] Validate token behavior under logging-only mode.
- [ ] Enforce Turnstile check validation by setting `TURNSTILE_ENFORCED=true`.

## 5. Supplier Packaging Mappings
- [ ] Create mapping forms on the POS dashboard for `SupplierProduct`.
- [ ] Implement the `PackagingConverter` utility class.
- [ ] Update the Purchase Order ingestion service to use `SupplierProduct` conversion ratios when adding stock.
- [ ] Verify that purchases correctly spawn batches with converted unit prices and base stock volumes.
