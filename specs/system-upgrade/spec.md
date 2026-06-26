# Functional Specification: System Upgrades (Locks, Inventory, Turnstile, Packaging)

## 1. Concurrency Locking Scenarios

### Scenario 1: Successful Lock Acquisition (Redis-based)
- **Given** a customer is requesting a lock on VIP Table 5 for `empresa_123`
- **When** the table is not currently locked by any other session
- **Then** the system must set a Redis key `tenant:empresa_123:lock:table:5` with a random token value and a TTL of 10 minutes
- **And** return a successful lock response to the client

### Scenario 2: Unsuccessful Lock Acquisition (Conflict)
- **Given** a customer is requesting a lock on VIP Table 5 for `empresa_123`
- **When** the table is already locked by another session token (e.g., the key `tenant:empresa_123:lock:table:5` exists)
- **Then** the system must reject the lock request with a status of `LOCKED_BY_OTHER`
- **And** must not overwrite the existing lock key

### Scenario 3: Atomic Lock Release
- **Given** a customer holds a valid lock token `token_abc` for VIP Table 5 under `empresa_123`
- **When** the customer submits a request to release the lock with token `token_abc`
- **Then** the system must execute the Lua script to verify that the value matches `token_abc`
- **And** delete the Redis key `tenant:empresa_123:lock:table:5`

### Scenario 4: Unauthorized Release Prevention
- **Given** a customer holds a lock token `token_abc` for VIP Table 5
- **When** another user tries to release the lock using token `token_xyz`
- **Then** the system must not delete the lock key
- **And** must return an error response

---

## 2. Batch/FIFO Inventory Scenarios

### Scenario 1: Purchase Order Creates Lotes
- **Given** a warehouse operator registers a purchase of 10 bottles of Vodka for `empresa_123` with batch code `LOT-101` and expiration date `2027-12-31`
- **When** the transaction is committed
- **Then** the system must create a record in `LoteProducto` with `cantidadOriginal = 10`, `cantidadRestante = 10`, and `fechaExpiracion = 2027-12-31`
- **And** write a `KardexLote` movement record of type `COMPRA`

### Scenario 2: Inventory Consumption using FEFO (First-Expired, First-Out)
- **Given** the system has two active batches for Vodka:
  - Batch A: `cantidadRestante = 4`, Expiration `2026-08-31`
  - Batch B: `cantidadRestante = 6`, Expiration `2026-12-31`
- **When** a bartender sells 5 bottles of Vodka in the POS using FEFO dispatch logic
- **Then** the system must fully consume Batch A (`cantidadRestante = 0`)
- **And** consume 1 bottle from Batch B (`cantidadRestante = 5`)
- **And** record two corresponding `KardexLote` records of type `VENTA`

### Scenario 3: Handling Expired Batches
- **Given** Batch A is expired (expiration date has passed)
- **When** a sale is processed
- **Then** the system must bypass Batch A
- **And** consume stock from the next unexpired batch according to the FEFO strategy

---

## 3. Turnstile Bot Verification Scenarios

### Scenario 1: Request with Valid Turnstile Token
- **Given** a booking request is received containing a Turnstile token `valid_token`
- **When** the system verifies the token against Cloudflare (`https://challenges.cloudflare.com/turnstile/v0/siteverify`) and receives `{ "success": true }`
- **Then** the system must process the reservation details
- **And** return a successful booking confirmation

### Scenario 2: Request with Invalid Turnstile Token
- **Given** a booking request is received containing an invalid Turnstile token or no token
- **When** the system verifies the token and receives `{ "success": false }`
- **Then** the system must block the reservation
- **And** return a `403 Forbidden` status code with the message `BOT_VERIFICATION_FAILED`

### Scenario 3: Turnstile Logging-Only Mode
- **Given** the environment variable `TURNSTILE_ENFORCED` is set to `false`
- **When** a request with an invalid Turnstile token is received
- **Then** the system must log a warning to the audit trail
- **And** allow the request to proceed and complete the booking

---

## 4. Supplier Packaging Mapping Scenarios

### Scenario 1: Box-to-Units Automatic Ingestion
- **Given** a supplier product mapping exists for "Vodka" with `skuProveedor = "VOD-BOX-12"`, `unidadEmbalaje = "CAJA"`, and `ratioConversion = 12`
- **When** the operator inputs a purchase of 3 boxes of `VOD-BOX-12`
- **Then** the system must calculate the base inventory quantity as `3 * 12 = 36` bottles
- **And** create a `LoteProducto` with `cantidadOriginal = 36` and `cantidadRestante = 36`
