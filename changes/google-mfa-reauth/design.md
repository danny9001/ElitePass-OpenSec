# Technical Design: Google Account MFA Setup Re-authentication and Passkey Direct Entry

## 1. Better-Auth Configuration Changes (`src/lib/auth.ts`)
We will configure the `twoFactor` plugin in Better-Auth to enable passwordless management. This allows users who signed up via social logins (Google) to enable and disable MFA without presenting a password.
```typescript
twoFactor({
  allowPasswordless: true,
})
```

## 2. Profile API/Page Changes (`src/app/dashboard/perfil/page.tsx`)
We need to know whether the user is a Google-linked account or a traditional credential-based account.
We will update the Prisma query to select the associated `accounts` for the current user and check if any account uses `providerId === "google"`.
```typescript
const user = await db.user.findUnique({
  where: { id: session.user.id },
  select: {
    id: true,
    name: true,
    apellido: true,
    email: true,
    image: true,
    telefono: true,
    ci: true,
    twoFactorEnabled: true,
    accounts: {
      select: {
        providerId: true,
      }
    }
  },
});
```
We will map this to `isGoogleAccount: user.accounts.some(acc => acc.providerId === "google")` and pass it to `PerfilClient`.

## 3. Client UI & Re-authentication Flow (`src/app/dashboard/perfil/PerfilClient.tsx`)
In `PerfilClient`, we will adjust both the MFA Enable and Disable flows.

### A. MFA Enable Flow (when `isGoogleAccount` is `true`):
- Instead of showing the password verification form (`mfaSetupStep === "password"`), we will display a verification prompt: *"Para configurar el doble factor en tu cuenta de Google, es necesario reautenticarte por seguridad."*
- A button *"Confirmar y Reautenticar con Google"* will initiate the sign-in redirect:
  ```typescript
  const callbackURL = `${window.location.origin}/dashboard/perfil?verifyMfa=google-enable`;
  await authClient.signIn.social({ provider: "google", callbackURL });
  ```
- Upon successful authentication, Google redirects back to `/dashboard/perfil?verifyMfa=google-enable`.
- An `useEffect` detects `verifyMfa === "google-enable"`, immediately calls `authClient.twoFactor.enable()` (without password), sets the state to `setup`, and retrieves the TOTP QR code and backup codes.

### B. MFA Disable Flow (when `isGoogleAccount` is `true`):
- Instead of asking for the user's password (`disableMfaStep === "password"`), we will display a verification prompt: *"Para desactivar el doble factor en tu cuenta de Google, es necesario reautenticarte por seguridad."*
- A button *"Confirmar y Reautenticar con Google"* will initiate the sign-in redirect:
  ```typescript
  const callbackURL = `${window.location.origin}/dashboard/perfil?verifyMfa=google-disable`;
  await authClient.signIn.social({ provider: "google", callbackURL });
  ```
- Since the user already has MFA enabled, Better-Auth will intercept this login: first requiring Google credentials, then redirecting them to `/login/mfa` to enter their MFA code.
- Once both are entered successfully, they redirect back to `/dashboard/perfil?verifyMfa=google-disable`.
- The `useEffect` detects `verifyMfa === "google-disable"`, immediately calls `authClient.twoFactor.disable()`, toggles `twoFactorEnabled` to false, and displays a success alert.

## 4. Passkey Direct Entry Compliance (`src/app/api/auth/webauthn/authenticate/route.ts`)
The custom WebAuthn authenticate route logs in the user and generates a Better-Auth session manually, bypassing the `twoFactor` plugin challenge. This is correct as Passkeys provide strong multi-factor authentication directly ("ingreso directo"), making further MFA checks redundant. No modifications are needed in `authenticate/route.ts`.
