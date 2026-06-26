# Design: QA Connection and Endpoint Verification Script

## Technical Approach

We will create two separate Node.js verification scripts (or a single script with sub-commands/steps) to test the database connections and API endpoints for both `elitepass-pos` and `elitepass-reservas`.
Since both applications use Prisma, we can load their respective environments, instantiate their Prisma clients, perform DB operations, and use `fetch` to request the API health endpoints.

### Project 1: `elitepass-pos` (POS System)
- **Root Path**: `/home/soporte/elitepass-pos/backend`
- **Prisma Schema**: `/home/soporte/elitepass-pos/backend/prisma/schema.prisma`
- **Database Env Variable**: `POS_DATABASE_URL` (loaded from `/home/soporte/elitepass-pos/backend/.env`)
- **Port**: `3001`
- **Tasks**:
  1. Instantiate `PrismaClient` using `POS_DATABASE_URL`.
  2. Fetch the first `Empresa` record: `prisma.empresa.findFirst()`.
  3. Fetch the first `Usuario` record: `prisma.usuario.findFirst()`.
  4. Create a mock product under this `Empresa`:
     - `nombre`: `"Test QA Beer"`
     - `categoria`: `"BEBIDA_ALCOHOLICA"`
     - `precioBase`: `15.0`
     - Clean up previous "Test QA Beer" products to ensure idempotency.
  5. Verify the created product is stored and queryable.
  6. Query `/api/v1/health` or `/api/health` on `http://localhost:3001` and verify it returns a 200 status code.

### Project 2: `elitepass-reservas` (Reservations System)
- **Root Path**: `/home/soporte/elitepass-reservas`
- **Prisma Schema**: `/home/soporte/elitepass-reservas/prisma/schema.prisma`
- **Database Env Variable**: `DATABASE_URL` (loaded from `/home/soporte/elitepass-reservas/.env`)
- **Port**: `3000`
- **Tasks**:
  1. Instantiate `PrismaClient` using `DATABASE_URL`.
  2. Fetch the first `Organization` record: `prisma.organization.findFirst()`.
  3. Fetch the first `User` record: `prisma.user.findFirst()`.
  4. Create a mock event under this `Organization`:
     - `name`: `"Test QA Club Night"`
     - `eventDate`: a week from today (e.g. `new Date(Date.now() + 7 * 24 * 60 * 60 * 1000)`)
     - `visibilityStart`: today (e.g. `new Date()`)
     - `visibilityEnd`: two weeks from today (e.g. `new Date(Date.now() + 14 * 24 * 60 * 60 * 1000)`)
     - Clean up previous "Test QA Club Night" events to ensure idempotency.
  5. Verify the created event is stored and queryable.
  6. Query `/api/health` or `/api/v1/health` on `http://localhost:3000` and verify it returns a 200 status code.

## Execution Tools
We can execute the script using `tsx` or `node` (with ES modules/MJS since they run node 18/20+).
Both projects have Node.js environments already set up, and they have `prisma` and `@prisma/client` installed. We can write scripts in the `/home/soporte/elitepass-pos/backend/scripts/` and `/home/soporte/elitepass-reservas/scripts/` directories respectively, or run a single runner from a central location.
Let's place the verification scripts directly in:
1. `/home/soporte/elitepass-pos/backend/scripts/qa-verify.mjs`
2. `/home/soporte/elitepass-reservas/scripts/qa-verify.mjs`
This isolates the dependencies and environment variables correctly within their corresponding directories.
