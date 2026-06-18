# Design: SSO Role Extraction and Cookie Signing

## Root Cause Analysis
1. **SSO Role Extraction Issue:**
   - The `/mi-cuenta` page in `elitepass-reservas` requires `role === "EXTERNAL"`.
   - External/customer users do not have a license for `reservas` in the identity system since they are public users. Therefore, `reservasApp` was undefined, defaulting to `"USER"`, which caused the database `upsert` to downgrade the user's role to `"USER"` and triggered redirection loops.
   - *Fix applied:* Extract `accountType` (`EXTERNAL`/`CLIENT`) and preserve roles to prevent downgrading.

2. **Session Cookie Signature Mismatch:**
   - Even when users logged in successfully, they were immediately redirected back to the login page.
   - *Analysis:* Better Auth expects cookies to be signed with the secret key (`BETTER_AUTH_SECRET`) using HMAC-SHA256:
     `cookieValue = `${sessionToken}.${signature}``
     where `signature` is the base64 HMAC-SHA256 signature of `sessionToken`.
   - The manual cookie writing in `identity-callback/route.ts` set the cookie to the plain text UUID `sessionToken` without a signature.
   - When the next request hit a protected route, Better Auth's `ctx.getSignedCookie()` failed to verify the signature of the session cookie and returned `null`.
   - Consequently, the session validator failed and redirected the user to `/sign-in` (which bounced to the Identity provider).

## Technical Decisions
1. **Extract `accountType` & Role Preservation:** (Completed in 1.4.82)
2. **Implement Cookie Signing:** Use the `{ makeSignature }` helper from `"better-auth/crypto"` to sign the session token cookie in `/api/auth/identity-callback` using `process.env.BETTER_AUTH_SECRET`.
3. **Set the Cookie correctly:** Write the signed token `${sessionToken}.${signature}` to `"__Secure-ea.session_token"` in production and `"ea.session_token"` in development.
