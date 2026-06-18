# Design: Payment Gateway Config Fix & Event QR Strategy

## Root Cause Analysis
1. **Validation Error on Gateway Save:** `elitepass-payments` strictly enforces `apiKey` existence on `POST /credentials` (upsert). However, the frontend sends an empty `apiKey` on subsequent updates (since the key is masked). The POS backend forwards this without an `apiKey`, causing a validation failure. Additionally, sending an empty `merchantCode` triggers a raw JSON error instead of a friendly UI message.
2. **Event Strategy Missing:** The database model `Evento` lacks a field to distinguish between using the automated payment gateway vs. manual QR processing. The UI currently assumes manual processing by default by listing manual bank accounts.

## Technical Decisions
1. **Gateway Configuration Fix:**
   - Update `backend/src/modules/payment-gateway/payment-gateway.service.ts` to use `PUT /credentials/:id` instead of `POST` if the config `id` exists. The PUT endpoint in the microservice allows partial updates.
   - Update `payment-gateway.router.ts` and `PaymentGatewayPage.tsx` to handle passing the `id`.
   - Add frontend validation to prevent empty `merchantCode` submissions.
2. **Event Payment Strategy Selection:**
   - Add a `usaPasarela Boolean @default(false)` field to the `Evento` Prisma schema.
   - Update `eventos.schema.ts`, `eventos.router.ts`, and `eventos.service.ts` to manage this field during Event CRUD operations.
   - Modify the `EventosPage.tsx` UI to present a toggle: "Pasarela de Pago Automatizada" vs "QR Manual (Cuentas Bancarias)".
   - In `PosPage.tsx`, if the active event has `usaPasarela = true`, initialize the payment form defaulting to `QR` (without a manual reference) so the backend triggers the automated payment gateway workflow.
