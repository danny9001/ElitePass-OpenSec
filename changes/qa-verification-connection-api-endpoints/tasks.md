# Tasks: QA Connection and Endpoint Verification Script

- [x] Investigate existing API health endpoints for POS and Reservas.
- [x] Create and run `qa-verify.mjs` script in `elitepass-pos/backend/scripts`.
  - [x] Connect to POS DB via Prisma.
  - [x] Query first `Empresa` and `Usuario`.
  - [x] Delete existing product named "Test QA Beer" (if any).
  - [x] Create new product "Test QA Beer" (category: "BEBIDA_ALCOHOLICA", precioBase: 15.0).
  - [x] Query and verify it exists.
  - [x] Query public API health endpoint.
- [x] Create and run `qa-verify.mjs` script in `elitepass-reservas/scripts`.
  - [x] Connect to Reservas DB via Prisma.
  - [x] Query first `Organization` and `User`.
  - [x] Delete existing event named "Test QA Club Night" (if any).
  - [x] Create new event "Test QA Club Night" (eventDate: 1 week from now, visibilityStart: now, visibilityEnd: 2 weeks from now).
  - [x] Query and verify it exists.
  - [x] Query public API health endpoint.
- [x] Record results in a final report/summary.
