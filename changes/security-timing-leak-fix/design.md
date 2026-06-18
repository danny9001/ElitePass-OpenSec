# Design: Constant-Time Token Verification

## Root Cause Analysis
In multiple files (`src/app/api/backups/run/route.ts`, `src/app/api/cron/archive/route.ts`, and `src/app/api/audit-logs/archive/route.ts`), authorization helper functions perform an early exit length-check (e.g. `left.length !== right.length` or `a.length === b.length`), which leaks the length of the secret token. In `src/app/api/internal/rate-limit/route.ts`, a direct non-constant-time string comparison (`authHeader !== \`Bearer \${secret}\``) is performed, which leaks the characters of the secret.
`timingSafeEqual` in Node.js throws an error if input lengths differ, so to do a constant-time comparison securely without throwing, both inputs must be normalized to a fixed length using a cryptographic hash function.

## Technical Decisions
To fix this timing leak, we will normalize the lengths of both inputs using a cryptographic hash function (SHA-256) before passing them to `timingSafeEqual`. This ensures that:
1. Both inputs to `timingSafeEqual` are always exactly 32 bytes.
2. The comparison time is completely constant, hiding the token's actual length and content.
3. No exceptions/errors are thrown during the comparison.

### Implementation Details:
```typescript
import { createHash, timingSafeEqual } from "node:crypto";

function isAuthorized(provided: string, secret: string): boolean {
  const hashA = createHash("sha256").update(provided).digest();
  const hashB = createHash("sha256").update(secret).digest();
  return timingSafeEqual(hashA, hashB);
}
```
This pattern will be applied to the authorization routines in the four affected routes.
