# Tasks: Payment Gateway Fix & Event QR Strategy

- [x] Modify `backend/src/modules/payment-gateway/payment-gateway.service.ts` to check for `data.id` and use `PUT /credentials/:id` for partial updates, or `POST` otherwise.
- [x] Modify `backend/src/modules/payment-gateway/payment-gateway.router.ts` to accept `id` in the JSON body.
- [x] Update `frontend/src/pages/PaymentGatewayPage.tsx` to send `id` in `saveMutation` and add `merchantCode` empty string validation.
- [x] Update `backend/prisma/schema.prisma` Event model with `usaPasarela Boolean @default(false)`.
- [x] Create and run a new Prisma migration for `usaPasarela`. (Done via db push to avoid drift conflicts)
- [x] Update `backend/src/modules/eventos/eventos.schema.ts` to include `usaPasarela`.
- [x] Update `backend/src/modules/eventos/eventos.router.ts` and `eventos.service.ts` to save and read `usaPasarela`.
- [x] Add `usaPasarela: boolean` to `Evento` interface in `frontend/src/types/index.ts`.
- [x] Update `frontend/src/pages/EventosPage.tsx` form UI to include the "Tipo de Cobro QR" selector (Pasarela Automatizada vs QR Manual).
- [x] Adjust `frontend/src/pages/PosPage.tsx` to pre-select Automated QR vs Manual QR based on `activeEvent?.usaPasarela`.
