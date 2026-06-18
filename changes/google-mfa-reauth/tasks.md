# Execution Tasks: Google Account MFA Setup Re-authentication and Passkey Direct Entry

## Phase 1: Better-Auth Config Update
- [x] Edit `src/lib/auth.ts` to set `allowPasswordless: true` in the `twoFactor` plugin settings.

## Phase 2: Profile Page Backend Update
- [x] Edit `src/app/dashboard/perfil/page.tsx` to include `accounts` selection in the Prisma user query.
- [x] Map the Prisma user result to include `isGoogleAccount: user.accounts.some(acc => acc.providerId === "google")`.

## Phase 3: Profile Page Client Component Update
- [x] Edit `src/app/dashboard/perfil/PerfilClient.tsx` to:
  - [x] Add `isGoogleAccount` to the `UserProfile` interface and `initialUser` destructured props.
  - [x] Implement a `useEffect` hook to intercept `verifyMfa` search query parameters (`google-enable` and `google-disable`).
  - [x] Upon detecting `verifyMfa === "google-enable"`, call `authClient.twoFactor.enable()` directly, clean the URL, and transition to the QR code setup step.
  - [x] Upon detecting `verifyMfa === "google-disable"`, call `authClient.twoFactor.disable()` directly, clean the URL, and reset the MFA state to disabled.
  - [x] Modify the MFA Enable password step UI: if the user is a Google account, display a re-authentication prompt and a button linking to Google sign-in instead of a password input.
  - [x] Modify the MFA Disable password step UI: if the user is a Google account, display a re-authentication prompt and a button linking to Google sign-in instead of a password input.

## Phase 4: Verification & Compilation
- [x] Test/compile the Next.js project using `pnpm run build` or similar compilation validation checks.
