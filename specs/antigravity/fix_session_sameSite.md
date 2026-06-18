---
name: fix-session-samesite-lax
description: Session cookie persistence fix — sameSite strict was breaking login
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 360d2505-3c6f-48c6-8c62-4743b54d19f2
---

**Problem:** After login at `/`, user was immediately logged out when page reloaded. The session cookie was not being preserved across page navigation.

**Root cause:** Cookie was configured with `sameSite: 'strict'` in `src/lib/auth.ts`. This is overly restrictive and prevents the cookie from being sent during page reloads and top-level navigations. The strict policy only allows cookies in same-site POST requests, not in navigation context.

**Solution:** Changed `sameSite: 'strict'` to `sameSite: 'lax'` in the `setSession()` function at `src/lib/auth.ts:82`.

`lax` allows cookies to be sent during:
- Top-level navigations (user clicking links)
- Page reloads (window.location.reload())
- Same-site GET requests

But still blocks cross-site requests (safer than 'none').

**How to apply:** If users report immediate logout after login, check the SameSite cookie policy. For this application, `lax` is appropriate since there are no cross-origin requests needed.

**Commit:** 341b4b8 - "Fix session cookie persistence — change sameSite from strict to lax"
