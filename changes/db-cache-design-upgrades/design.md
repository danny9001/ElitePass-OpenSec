# Technical Design: Database, Caching, and Pooling Upgrades

This document details the database schema additions, Redis-based distributed locking schemas, and connection pool configurations required to support high-concurrency event operations, bot mitigation, and audit-compliant FIFO/Lote inventory tracking.

---

## 1. Schema Extensions in `schema.prisma`

To upgrade `elitepass-pos` and `elitepass-reservas` without breaking existing database tables, we introduce new models as non-breaking extensions. These models establish relations to existing structures (`Producto`, `Empresa`, `Proveedor`, `Evento`, `MovimientoStock`).

### 1.1. FIFO/FEFO Batch Tracking (POS System)

We add `LoteProducto` (to track stock batches with expiration dates and individual costs) and `KardexLote` (an immutable ledger linking stock movements to specific batches).

```prisma
// =============================================================================
// BATCH STOCK TRACKING (LOTES / FIFO / FEFO)
// =============================================================================

model LoteProducto {
  id               String       @id @default(cuid())
  empresaId        String       @map("empresa_id")
  eventoId         String?      @map("evento_id")
  productoId       String       @map("producto_id")
  codigoLote       String       @map("codigo_lote")      // e.g. "LOT-2026-06-A"
  cantidadOriginal Float        @map("cantidad_original") // Initial amount in base units
  cantidadRestante Float        @map("cantidad_restante") // Current remaining amount in base units
  costoUnitario    Float        @map("costo_unitario")    // Unit cost at ingestion
  fechaExpiracion  DateTime?    @map("fecha_expiracion")
  fechaIngreso     DateTime     @default(now()) @map("fecha_ingreso")
  creadoEn         DateTime     @default(now()) @map("creado_en")
  actualizadoEn    DateTime     @updatedAt @map("actualizado_en")

  // Relations
  empresa          Empresa      @relation(fields: [empresaId], references: [id], onDelete: Cascade)
  evento           Evento?      @relation(fields: [eventoId], references: [id], onDelete: SetNull)
  producto         Producto     @relation(fields: [productoId], references: [id], onDelete: Cascade)
  movimientosLote  KardexLote[]

  @@unique([empresaId, productoId, codigoLote])
  @@index([empresaId, productoId, fechaExpiracion])
  @@index([empresaId, productoId, fechaIngreso])
  @@index([eventoId])
  @@map("lotes_producto")
}

model KardexLote {
  id                String          @id @default(cuid())
  loteId            String          @map("lote_id")
  movimientoStockId String?         @map("movimiento_stock_id")
  cantidad          Float           // Negative for consumption, positive for additions
  creadoEn          DateTime        @default(now()) @map("creado_en")

  // Relations
  lote              LoteProducto    @relation(fields: [loteId], references: [id], onDelete: Cascade)
  movimientoStock   MovimientoStock? @relation(fields: [movimientoStockId], references: [id], onDelete: Cascade)

  @@index([loteId])
  @@index([movimientoStockId])
  @@map("kardex_lotes")
}
```

#### Backwards-Compatibility Strategy
- Existing tables like `StockAlmacen` and `StockBarra` will continue to store aggregate totals.
- A background reconciliation system writes to both models. When reading stock, the system uses the sum of `LoteProducto.cantidadRestante` for active batches. If a fallback is needed, it checks the aggregate.

---

### 1.2. Supplier Packaging Mappings

To map vendor packaging sizes (e.g., box of 12 bottles, 50L keg) and auto-convert them to base inventory units, we introduce the `SupplierProduct` mapping table:

```prisma
// =============================================================================
// SUPPLIER PACKAGING MAPPING & UNIT CONVERSIONS
// =============================================================================

model SupplierProduct {
  id              String   @id @default(cuid())
  empresaId       String   @map("empresa_id")
  proveedorId     String   @map("proveedor_id")
  productoId      String   @map("producto_id")
  skuProveedor    String   @map("sku_proveedor")      // Vendor SKU code
  precioCompra    Float    @map("precio_compra")      // Cost per package unit
  unidadEmbalaje  String   @map("unidad_embalaje")    // e.g. "CAJA", "BARRIL", "PACK_6"
  ratioConversion Float    @map("ratio_conversion")   // Factor to convert packaging to base unit (e.g. 12.0)
  creadoEn        DateTime @default(now()) @map("creado_en")
  actualizadoEn   DateTime @updatedAt @map("actualizado_en")

  // Relations
  empresa         Empresa   @relation(fields: [empresaId], references: [id], onDelete: Cascade)
  proveedor       Proveedor @relation(fields: [proveedorId], references: [id], onDelete: Cascade)
  producto        Producto  @relation(fields: [productoId], references: [id], onDelete: Cascade)

  @@unique([empresaId, proveedorId, skuProveedor])
  @@index([empresaId, productoId])
  @@map("supplier_products")
}
```

#### Schema Relations Additions to Existing Parent Models:
To satisfy Prisma compilation rules, the parent models must define their back-relations:
```prisma
// Inside model Empresa:
// lotesProducto     LoteProducto[]
// supplierProducts  SupplierProduct[]

// Inside model Producto:
// lotesProducto     LoteProducto[]
// supplierProducts  SupplierProduct[]

// Inside model Proveedor:
// supplierProducts  SupplierProduct[]

// Inside model Evento:
// lotesProducto     LoteProducto[]

// Inside model MovimientoStock:
// movementsLote      KardexLote[]
```

---

## 2. Redis Caching & Lock Keys Design

Under high-concurrency ticket releases, PostgreSQL transactional locks on `ReservationLock` block DB connections. In-memory locks via Redis avoid this bottleneck.

### 2.1. Lock Key & Cache Key Structure (Multi-Tenant Namespacing)

To ensure strict tenant isolation, all Redis keys follow a colon-separated namespace structure:

| Key Type | Structure Pattern | Example Key |
|---|---|---|
| **Table Lock** | `tenant:{empresaId}:lock:table:{tableId}` | `tenant:cltd123:lock:table:tbl987` |
| **Ticket Lock** | `tenant:{empresaId}:lock:ticket_stage:{stageId}` | `tenant:cltd123:lock:ticket_stage:stg555` |
| **Event Cache** | `tenant:{empresaId}:cache:event:{eventId}:details` | `tenant:cltd123:cache:event:evt444:details` |
| **User Session**| `tenant:{empresaId}:cache:session:{token}` | `tenant:cltd123:cache:session:tkn_9999...` |

### 2.2. Distributed Locking Protocol

#### Lock Acquisition
Locks are acquired atomically using the Redis `SET` command with parameters `NX` (Not Exists) and `PX` (TTL in milliseconds):

```bash
SET tenant:cltd123:lock:table:tbl987 "unique_client_session_uuid" NX PX 300000
```
- **NX**: Ensures the lock is acquired only if it does not already exist.
- **PX 300000**: Expires the lock in 5 minutes (300,000ms) to prevent deadlocks in case of client crashes.
- **unique_client_session_uuid**: A unique token (e.g., UUIDv4) generated by the requesting client. This prevents a client from accidentally releasing a lock acquired by a subsequent reservation request.

#### Lock Release (Atomic Lua Script)
To release the lock safely, the client executes a Lua script via `EVAL`. This script compares the value of the key to the client's UUID before deleting:

```lua
if redis.call("get", KEYS[1]) == ARGV[1] then
    return redis.call("del", KEYS[1])
else
    return 0
end
```
**Redis CLI execution example:**
```bash
EVAL "if redis.call('get', KEYS[1]) == ARGV[1] then return redis.call('del', KEYS[1]) else return 0 end" 1 tenant:cltd123:lock:table:tbl987 "unique_client_session_uuid"
```

### 2.3. Caching Patterns & TTLs

- **Active Event Catalog**: Saved under `tenant:{empresaId}:cache:events:active`.
  - **TTL**: 10 minutes (600s).
  - **Eviction Strategy**: Purge cache key whenever an admin edits/activates an event.
- **Ticket Stock Counts**: Saved under `tenant:{empresaId}:cache:ticket_stage:{stageId}:stock`.
  - **TTL**: 30 seconds (high-concurrency path).
  - **Update Policy**: Read from cache, fallback to Postgres on miss, invalidate on purchase.

### 2.4. Resiliency & Postgres Fallback (Circuit Breaker)
- If Redis is unreachable, the system triggers a circuit breaker and falls back to SQL database locking (`ReservationLock` table).
- Setting `USE_REDIS_LOCKS=false` in environment variables disables Redis locking entirely, reverting to standard Postgres locks.

---

## 3. PgBouncer Tuning under Heavy Concurrency / Bot Floods

Bots targeting ticketing endpoints can exhaust database connection pools in seconds. PgBouncer must be tuned to absorb connection spikes and fail fast rather than stalling the application.

### 3.1. Pool Mode Selection: Transaction Mode
- **Mode**: `pool_mode = transaction` is **mandatory** for high-concurrency Node.js apps connected to PostgreSQL.
- **Why**: Allows connections to be shared and reused at the boundary of each transaction/query, maximizing throughput (hundreds of client connections mapped to a few dozen Postgres backends).
- **Prisma Caveats**: Prisma uses prepared statements, which do not natively work in PgBouncer transaction mode. To resolve this, append `?pgbouncer=true` to the database URL in the `.env` file. This tells Prisma to execute raw queries or use a custom prepared statement caching proxy.

### 3.2. Recommended `pgbouncer.ini` Configurations

```ini
[databases]
# Append pgbouncer=true in Prisma's connection string
elitepass_db = host=127.0.0.1 port=5432 dbname=elitepass_prod auth_user=postgres

[pgbouncer]
logfile = /var/log/postgresql/pgbouncer.log
pidfile = /var/run/postgresql/pgbouncer.pid
listen_addr = 0.0.0.0
listen_port = 6432
auth_type = md5
auth_file = /etc/pgbouncer/userlist.txt
pool_mode = transaction

# =============================================================================
# CONCURRENCY AND POOL SIZE TUNING
# =============================================================================

# Maximum client connections PgBouncer will accept (limits socket allocation)
max_client_conn = 10000

# Base pool size per database/user combination
default_pool_size = 150

# Minimum server connections to keep open to reduce TCP handshake overhead
min_pool_size = 30

# Emergency overflow connection pool when default pool size is exceeded
reserve_pool_size = 50

# How long a client waits in the queue before reserve connections are opened
reserve_pool_timeout = 2

# Max server connections allowed across all databases
max_db_connections = 250

# =============================================================================
# TIMEOUTS AND BOT MITIGATION (FAIL FAST)
# =============================================================================

# Close client connection if authentication takes longer than 5 seconds (blocks idle bots)
client_login_timeout = 5

# Close server connections idle for 120s to reclaim OS file descriptors
server_idle_timeout = 120

# Force reconnection after 1 hour to prevent memory fragmentation
server_lifetime = 3600

# Close server connections in transaction mode if query takes > 10 seconds (kills hung transactions)
query_timeout = 10

# Timeout for connection establishment to PostgreSQL backend
server_connect_timeout = 10
```

### 3.3. PostgreSQL Engine-Level Configurations
To prevent long-running queries from holding PgBouncer connections hostaged:
1. **Set Statement Timeout**:
   ```sql
   ALTER DATABASE elitepass_prod SET statement_timeout = '10s';
   ```
2. **Configure Idle in Transaction Timeout**:
   ```sql
   ALTER DATABASE elitepass_prod SET idle_in_transaction_session_timeout = '5s';
   ```
3. **Index Foreign Keys**: Ensure all foreign keys (`eventId`, `tableId`, `productoId`) have composite indexes to speed up lookup times, preventing pool starvation during mass deletes or updates.
