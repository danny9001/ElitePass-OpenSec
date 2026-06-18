# Design: Multi-Tenant Payment Methods

## Database Schema Additions

1. **`CuentaBancariaEmpresa`** (Company Bank Account)
   - Belongs to an `Empresa`.
   - Fields: `id`, `empresaId`, `banco` (e.g., 'BCP', 'BNB', 'PLATAFORMA'), `nombreCuenta`, `numeroCuenta`, `tipoCuenta` (e.g., Ahorro, Corriente, QR_MANUAL), `titular`, `documentoIdentidad`, `qrImageUrl`, `activa`.
   - Enables companies to register multiple accounts. If a company doesn't register one, the system can provide a platform default account via logic.

2. **`EventoCuentaBancaria`** (Event Payment Account Settings)
   - Joins `Evento` and `CuentaBancariaEmpresa`.
   - Fields: `id`, `eventoId`, `cuentaBancariaId`, `esPorDefecto` (Boolean).
   - Allows users to select which of their company accounts are active for a specific event, and which one is the default payment method for reservations and POS.

3. **Changes to `Evento`**:
   - Relationship `cuentasBancarias EventoCuentaBancaria[]`

## Technical Decisions
- **Microservice / Backend**: We will create REST endpoints (or update existing Evento/Empresa endpoints) to manage `CuentaBancariaEmpresa`.
- **Event Creation/Edition**: The payload for creating/editing an event will now accept an array of `cuentasBancarias` with their `esPorDefecto` flags.
- **Reservations & POS Default**: The frontend will query the active accounts for the selected event. The one with `esPorDefecto = true` will be pre-selected in the UI for QR or bank transfer payments.

## Root Cause Analysis
Currently, payment logic is hardcoded or relies on generic `MetodoPago` enums (`EFECTIVO`, `TARJETA`, `QR`). There is no tie-in to actual bank accounts (BNB, BCP) at the Empresa or Evento level, which makes it impossible to dynamically route payments or display correct QRs per company/event.
