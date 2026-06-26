# Tasks: API Security & Bot Mitigation Implementation

- [x] **Step 1: Redis sliding window implementation**
  - [x] Define Redis connection configuration and export client helper if missing.
  - [x] Implement atomic sliding window rate limiter in `src/lib/security/rate-limit.ts` using Redis sorted sets (or via `/api/internal/rate-limit` endpoint).
  - [x] Add fallback mechanism to standard key-value memory store if Redis connectivity drops.

- [x] **Step 2: Turnstile server-side validation**
  - [x] Create a reusable Turnstile validation function in `src/lib/security/turnstile.ts`.
  - [x] Integrate verification function into authentication and reservation Server Actions (e.g. `submitReservationAction`).
  - [x] Implement user-friendly error returns when Turnstile verification fails.

- [x] **Step 3: Headless browser & device detection**
  - [x] Define bot User-Agent blacklists and browser inconsistency heuristics.
  - [x] Implement header checks (`sec-ch-ua`, `accept-language`, `webdriver`) in a middleware helper function.
  - [x] Add bot telemetry checking endpoint or client-side checks inside high-risk pages.

- [x] **Step 4: Middleware Integration & Verification**
  - [x] Update `src/middleware.ts` to execute bot checks and apply sliding window rate limiting.
  - [x] Return appropriate status codes: `403 Forbidden` for bot signatures, `429 Too Many Requests` with `Retry-After` header for rate limiters.
  - [x] Validate implementation behavior.

