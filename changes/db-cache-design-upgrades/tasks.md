# Tasks: Database, Caching, and PgBouncer Tuning Implementation

## 1. Schema Extensions (`schema.prisma`)
- [x] Incorporate `LoteProducto` and `KardexLote` models into POS schema.prisma.
- [x] Add back-relations in parent models (`Empresa`, `Producto`, `Evento`, `MovimientoStock`).
- [x] Incorporate `SupplierProduct` model into POS schema.prisma.
- [x] Add back-relations for `SupplierProduct` in `Empresa`, `Producto`, and `Proveedor`.
- [x] Run a schema validation check using `prisma validate`.

## 2. Redis Caching & Lock Setup
- [ ] Set up client connection pooling for Redis (`ioredis` or `node-redis`) inside infrastructure layers of both applications.
- [ ] Implement atomic lock acquisition using `SET key token NX PX ttl`.
- [ ] Deploy the Lua script for atomic safe lock releases.
- [ ] Implement multi-tenant namespace helpers `tenant:{empresaId}:lock:...` and `tenant:{empresaId}:cache:...`.
- [ ] Add the circuit breaker pattern to fallback to standard SQL database `ReservationLock` table on Redis disconnection.

## 3. PgBouncer Optimization
- [ ] Apply connection string parameters `?pgbouncer=true` to all application database URL environment variables.
- [ ] Update `pgbouncer.ini` with high-concurrency pool values (`max_client_conn = 10000`, `default_pool_size = 150`, etc.).
- [ ] Set `query_timeout = 10` and `client_login_timeout = 5` in `pgbouncer.ini` for bot mitigation.
- [ ] Run database adjustments: `ALTER DATABASE ... SET statement_timeout = '10s'` and `idle_in_transaction_session_timeout = '5s'`.
- [ ] Confirm composite indexes exist on lock and transaction tables.
