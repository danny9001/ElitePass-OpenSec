# Design: Audit Reservations Application and Update Global Skills

## Audit Findings - Hot Code Extraction

### 1. Core Security
- **Authentication & SSO Integration**:
  - Configured with `better-auth` using Prisma Adapter on PostgreSQL.
  - Local credentials login and password endpoints (`/api/auth/sign-in`, `/api/auth/sign-up`, etc.) are blocked with 403 Forbidden to force SSO redirection to ElitePass Identity (`id.genial-it.net`).
  - Session TTL is set to 12 hours (`expiresIn: 60 * 60 * 12`) and utilizes secure, HTTP-only, Strict same-site cookies prefixed with `ea`.
- **IP Detection**:
  - Resolves client IP dynamically from headers: `cf-connecting-ip` -> `x-real-ip` -> `x-forwarded-for` (first entry in list).
- **CORS Handling**:
  - Dynamically allows origins from environment `ALLOWED_ORIGINS` (comma-separated list), `NEXT_PUBLIC_APP_URL`, `BETTER_AUTH_URL`, `NEXT_PUBLIC_BETTER_AUTH_URL`, and the current request origin.
  - Automatically handles preflight OPTIONS requests, returning 204 No Content with CORS headers.
- **CSRF Protection**:
  - POST, PUT, PATCH, DELETE mutations are checked against allowed origins and referers.
  - Routes like `/api/package-reservations/`, `/api/approvals/`, `/api/uploads/`, `/api/exports/`, `/api/guests/`, and `/api/qr/` strictly require `x-csrf-check: 1` header.
- **Payload/Content-Length Limits**:
  - Content-length is validated in the middleware:
    - `/api/uploads/`: 10 MB limit
    - `/api/users/avatar`: 6 MB limit
    - `/api/auth/`: 100 KB limit
    - All other endpoints: 1 MB limit
  - Exceeding payloads return 413 "Payload too large".
- **File Validation & Magic Bytes**:
  - Magic byte validation checks the first bytes of uploaded files to prevent mime-type spoofing:
    - `image/jpeg` -> `[0xff, 0xd8, 0xff]`
    - `image/png` -> `[0x89, 0x50, 0x4e, 0x47]`
    - `image/webp` -> `[0x52, 0x49, 0x46, 0x46]`
    - `application/pdf` -> `[0x25, 0x50, 0x44, 0x46]`
  - Explicitly blocks extensions: `js`, `mjs`, `cjs`, `ts`, `tsx`, `php`, `php3`, `php4`, `php5`, `phtml`, `exe`, `dll`, `com`, `bat`, `cmd`, `sh`, `bash`, `zsh`, `fish`, `py`, `rb`, `pl`, `lua`, `jar`, `war`, `ear`, `asp`, `aspx`, `jsp`, `htaccess`, `htpasswd`.
  - Filenames are sanitized via `sanitizeFilename` by replacing directory traversals and non-alphanumeric characters with `_`.
- **Input Sanitization**:
  - `sanitizeInput(value, maxLength)` strips HTML tags (`<[^>]*>`), trims, and escapes special characters (`&`, `<`, `>`, `"`, `'`, `/`, `` ` ``) into HTML entities.
  - Integrated directly inside Zod schemas (e.g. `packageReservationGuestSchema`) using `.transform()`.
- **Security Headers**:
  - Strict-Transport-Security: `max-age=63072000; includeSubDomains; preload` (in production).
  - X-Frame-Options: `DENY`
  - X-Content-Type-Options: `nosniff`
  - Referrer-Policy: `strict-origin-when-cross-origin`
  - Permissions-Policy: `camera=(self), microphone=(), geolocation=(), payment=(), usb=()`
  - Content-Security-Policy (CSP): uses a dynamic UUID-based nonce injected per request.

### 2. Core Events & Backend
- **Deferred Audit Logging Queue**:
  - Safe writes are deferred in-process via `audit-queue.ts` which buffers events in memory and flushes them every 500ms using `db.auditLog.createMany` (splicing 100 entries per batch with `skipDuplicates: true`).
  - Active Prisma Transactions bypass the queue and log synchronously using the transaction client to ensure consistency.
- **Log Sanitization**:
  - Logs are parsed through `sanitizeLogValue`, which masks email strings (e.g. `j***@domain.com`), tokens/secrets, and replaces any object keys matching `/(password|token|secret|authorization)/i` with `***`.
- **Rate-Limiting Storage**:
  - Dual-mode rate limit checking: posts to the database-backed endpoint `/api/internal/rate-limit` using a shared authorization header secret (`RATE_LIMIT_INTERNAL_SECRET` or `BACKUP_CRON_SECRET`), and falls back to a global in-memory Map (`globalThis.__edgeRateLimitStore`) in Next.js Edge.
  - The database check executes a Prisma interactive `$transaction` to fetch/upsert `RateLimitBucket` records.
- **Azure Cloud Storage**:
  - Integrates `@azure/storage-blob` utilizing `AZURE_STORAGE_CONNECTION_STRING` and `AZURE_STORAGE_CONTAINER_NAME`.
  - Images are normalized on upload to WebP using `sharp` (quality 95/alpha 100 for JPEGs, lossless for PNGs, and thumbnail 200x200 quality 40 for Avatars).
  - Blob paths are structured as: `folder/YYYY/MM/entity-slug/YYYYMMDD-entity-slug-uid.webp`.
- **Deduplication & Transaction Tracing**:
  - Bulk operations (such as batch invitations) generate a transaction correlation ID using `crypto.randomUUID()` and link all entries under this `requestId`.
  - Database mutations are guarded by Prisma `$transaction` blocks to roll back changes automatically on failure.

### 3. Core Design System
- **OKLCH Palette**:
  - Tailwind CSS v4 is configured with system colors defined in `globals.css` using `oklch` syntax:
    - Background: `oklch(1 0 0)` (light) / `oklch(0.145 0 0)` (dark)
    - Foreground: `oklch(0.145 0 0)` (light) / `oklch(0.985 0 0)` (dark)
    - Card: `oklch(1 0 0)` (light) / `oklch(0.205 0 0)` (dark)
    - Primary: `oklch(0.205 0 0)` (light) / `oklch(0.922 0 0)` (dark)
    - Border: `oklch(0.922 0 0)` (light) / `oklch(1 0 0 / 10%)` (dark)
- **Visual Density & Typography**:
  - Sans-serif Font: `"Segoe UI", "Helvetica Neue", Helvetica, Arial, sans-serif`
  - Mono Font: `"SFMono-Regular", Consolas, "Liberation Mono", Menlo, monospace`
  - Icon Webfont: `Material Symbols Outlined` loaded from local `/fonts/material-symbols-outlined.ttf`.
  - Border Radii: base `--radius` is `0.625rem` (10px). Derived radii: `radius-sm` (6px), `radius-md` (8px), `radius-lg` (10px), `radius-xl` (14px), `radius-2xl` (18px).
- **Responsive Layout & PWA notches**:
  - Uses default Tailwind screens.
  - Safe Areas padding offsets: `.system-layout` (`safe-area-inset-top`, `safe-area-inset-left`, `safe-area-inset-right`), `.system-bottom-nav` (`safe-area-inset-bottom`).
  - Standalone PWA padding offsets: `.system-content` has `padding-bottom: calc(5.5rem + env(safe-area-inset-bottom))` in standalone mode, and `body.mobile-bottom-nav .system-content` has `padding-bottom: calc(5.75rem + env(safe-area-inset-bottom))` on mobile.
- **Preference Persistence**:
  - Uses `next-themes` (persisted in localStorage key `theme`).
  - Personalization for PWA Virtual Credential (`VirtualCredentialPreference` DB model) persists user card preferences (custom company names, custom logos uploaded to Azure Blob, and colors: `backgroundPrimaryColor`, `backgroundSecondaryColor`, `textColor`).
  - LocalStorage stores: `pwa-dismissed` (install prompts), `stats-mode:${storageKey}` (dashboard charts state), and `pwa-credential-data` (cached virtual card payload).
