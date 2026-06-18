# Tasks: Multi-Tenant Payment Methods

- [x] Update `backend/prisma/schema.prisma` with `CuentaBancariaEmpresa` and `EventoCuentaBancaria` models.
- [x] Run Prisma migration (done via db push previously).
- [x] Backend: Create CRUD endpoints for `CuentaBancariaEmpresa` (Company Payment Accounts).
- [x] Backend: Update Event creation/edition endpoints to accept and save `EventoCuentaBancaria` associations (and validate `esPorDefecto`).
- [x] Backend: Update Event fetching logic to include associated payment accounts.
- [x] Frontend: Create a UI in the Company settings to add/edit payment accounts (BNB, BCP, manual QR).
- [x] Frontend: Update the Event creation/edition form to include a "Payment Methods" section, allowing selection of accounts and marking one as default.
- [x] Frontend (Reservations & POS): Retrieve the Event's default payment method and pre-select it in the checkout/payment UI.
